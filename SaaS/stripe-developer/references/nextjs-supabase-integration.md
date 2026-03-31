# Next.js + Supabase + Vercel Integration

This is the cross-stack wiring guide. It covers how Stripe connects to the rest of the SaaS stack: Next.js (frontend + server logic), Supabase (auth + database + RLS), and Vercel (deployment + env vars).

## The Pattern

```
User clicks "Subscribe"
  → Next.js Server Action creates a Stripe Checkout Session
  → User pays on Stripe Checkout
  → Stripe fires webhook to your Next.js Route Handler
  → Route Handler updates subscription status in Supabase
  → RLS policies gate access based on subscription status
  → User sees pro features
```

This is the payment flow for every SaaS built on this stack.

## Supabase Database Schema

These tables mirror Stripe's data model. Sync them via webhooks so you never need to query Stripe on every request.

```sql
-- Users table (extends Supabase auth.users)
create table public.users (
  id uuid references auth.users not null primary key,
  full_name text,
  email text,
  stripe_customer_id text unique,
  created_at timestamptz default now()
);

alter table public.users enable row level security;
create policy "Users can view own data" on public.users
  for select to authenticated
  using ((select auth.uid()) = id);
create policy "Users can update own data" on public.users
  for update to authenticated
  using ((select auth.uid()) = id);

-- Products (synced from Stripe)
create table public.products (
  id text primary key, -- Stripe product ID (prod_xxx)
  name text,
  description text,
  active boolean default true,
  metadata jsonb
);

alter table public.products enable row level security;
create policy "Products are viewable by everyone" on public.products
  for select using (true);

-- Prices (synced from Stripe)
create type pricing_type as enum ('one_time', 'recurring');
create type pricing_interval as enum ('day', 'week', 'month', 'year');

create table public.prices (
  id text primary key, -- Stripe price ID (price_xxx)
  product_id text references public.products(id),
  active boolean default true,
  description text,
  unit_amount bigint,
  currency text check (char_length(currency) = 3),
  type pricing_type,
  interval pricing_interval,
  interval_count integer,
  trial_period_days integer,
  metadata jsonb
);

alter table public.prices enable row level security;
create policy "Prices are viewable by everyone" on public.prices
  for select using (true);

-- Subscriptions (synced from Stripe)
create type subscription_status as enum (
  'trialing', 'active', 'canceled', 'incomplete',
  'incomplete_expired', 'past_due', 'unpaid', 'paused'
);

create table public.subscriptions (
  id text primary key, -- Stripe subscription ID (sub_xxx)
  user_id uuid references auth.users(id) not null,
  status subscription_status,
  price_id text references public.prices(id),
  quantity integer,
  cancel_at_period_end boolean default false,
  current_period_start timestamptz,
  current_period_end timestamptz,
  created_at timestamptz default now(),
  ended_at timestamptz,
  cancel_at timestamptz,
  canceled_at timestamptz,
  trial_start timestamptz,
  trial_end timestamptz,
  metadata jsonb
);

alter table public.subscriptions enable row level security;
create policy "Users can view own subscriptions" on public.subscriptions
  for select to authenticated
  using ((select auth.uid()) = user_id);

-- Index for fast lookups
create index idx_subscriptions_user_id on public.subscriptions(user_id);
create index idx_subscriptions_status on public.subscriptions(status);
```

## Webhook Handler with Supabase Sync

This is the complete webhook handler that syncs Stripe events into your Supabase database.

```ts
// app/api/stripe/webhook/route.ts
import { headers } from 'next/headers'
import { NextResponse } from 'next/server'
import Stripe from 'stripe'
import { stripe } from '@/lib/stripe'
import { createClient } from '@supabase/supabase-js'

// Use the service role key for webhook handlers — they run without a user session
const supabaseAdmin = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.SUPABASE_SERVICE_ROLE_KEY!
)

export async function POST(request: Request) {
  const body = await request.text()
  const headersList = await headers()
  const signature = headersList.get('stripe-signature')!

  let event: Stripe.Event
  try {
    event = stripe.webhooks.constructEvent(
      body,
      signature,
      process.env.STRIPE_WEBHOOK_SECRET!
    )
  } catch (err) {
    console.error('Webhook signature verification failed:', err)
    return NextResponse.json({ error: 'Invalid signature' }, { status: 400 })
  }

  try {
    switch (event.type) {
      // ── Product & Price sync ──────────────────────────
      case 'product.created':
      case 'product.updated': {
        const product = event.data.object as Stripe.Product
        await supabaseAdmin.from('products').upsert({
          id: product.id,
          name: product.name,
          description: product.description,
          active: product.active,
          metadata: product.metadata,
        })
        break
      }

      case 'price.created':
      case 'price.updated': {
        const price = event.data.object as Stripe.Price
        await supabaseAdmin.from('prices').upsert({
          id: price.id,
          product_id: typeof price.product === 'string' ? price.product : price.product.id,
          active: price.active,
          description: price.nickname,
          unit_amount: price.unit_amount,
          currency: price.currency,
          type: price.type,
          interval: price.recurring?.interval ?? null,
          interval_count: price.recurring?.interval_count ?? null,
          trial_period_days: price.recurring?.trial_period_days ?? null,
          metadata: price.metadata,
        })
        break
      }

      // ── Checkout completed ────────────────────────────
      case 'checkout.session.completed': {
        const session = event.data.object as Stripe.Checkout.Session
        if (session.mode === 'subscription') {
          const subscription = await stripe.subscriptions.retrieve(
            session.subscription as string
          )
          // Link Stripe customer to your user
          // Assumes you pass user_id in session metadata
          const userId = session.metadata?.user_id
          if (userId) {
            await supabaseAdmin.from('users').update({
              stripe_customer_id: session.customer as string,
            }).eq('id', userId)

            await upsertSubscription(subscription, userId)
          }
        }
        break
      }

      // ── Subscription lifecycle ────────────────────────
      case 'customer.subscription.created':
      case 'customer.subscription.updated': {
        const subscription = event.data.object as Stripe.Subscription
        const userId = await getUserIdFromCustomer(subscription.customer as string)
        if (userId) await upsertSubscription(subscription, userId)
        break
      }

      case 'customer.subscription.deleted': {
        const subscription = event.data.object as Stripe.Subscription
        const userId = await getUserIdFromCustomer(subscription.customer as string)
        if (userId) {
          await supabaseAdmin.from('subscriptions').update({
            status: 'canceled',
            ended_at: new Date(subscription.ended_at! * 1000).toISOString(),
          }).eq('id', subscription.id)
        }
        break
      }

      // ── Invoice events ────────────────────────────────
      case 'invoice.paid': {
        const invoice = event.data.object as Stripe.Invoice
        if (invoice.subscription) {
          const subscription = await stripe.subscriptions.retrieve(
            invoice.subscription as string
          )
          const userId = await getUserIdFromCustomer(invoice.customer as string)
          if (userId) await upsertSubscription(subscription, userId)
        }
        break
      }

      case 'invoice.payment_failed': {
        const invoice = event.data.object as Stripe.Invoice
        // TODO: Notify user (email, in-app banner)
        console.log(`Payment failed for customer ${invoice.customer}`)
        break
      }

      default:
        console.log(`Unhandled event: ${event.type}`)
    }
  } catch (err) {
    console.error('Webhook processing error:', err)
    return NextResponse.json({ error: 'Handler failed' }, { status: 500 })
  }

  return NextResponse.json({ received: true })
}

// ── Helper functions ──────────────────────────────────

async function upsertSubscription(subscription: Stripe.Subscription, userId: string) {
  const item = subscription.items.data[0]
  await supabaseAdmin.from('subscriptions').upsert({
    id: subscription.id,
    user_id: userId,
    status: subscription.status,
    price_id: item.price.id,
    quantity: item.quantity ?? 1,
    cancel_at_period_end: subscription.cancel_at_period_end,
    current_period_start: new Date(subscription.current_period_start * 1000).toISOString(),
    current_period_end: new Date(subscription.current_period_end * 1000).toISOString(),
    canceled_at: subscription.canceled_at
      ? new Date(subscription.canceled_at * 1000).toISOString()
      : null,
    trial_start: subscription.trial_start
      ? new Date(subscription.trial_start * 1000).toISOString()
      : null,
    trial_end: subscription.trial_end
      ? new Date(subscription.trial_end * 1000).toISOString()
      : null,
    metadata: subscription.metadata,
  })
}

async function getUserIdFromCustomer(stripeCustomerId: string): Promise<string | null> {
  const { data } = await supabaseAdmin
    .from('users')
    .select('id')
    .eq('stripe_customer_id', stripeCustomerId)
    .single()
  return data?.id ?? null
}
```

## Gating Features with RLS

Once subscription status is in your database, use RLS to gate access:

```sql
-- Example: Only active subscribers can access premium content
create policy "Pro users can access premium content" on public.premium_content
  for select to authenticated
  using (
    exists (
      select 1 from public.subscriptions
      where subscriptions.user_id = (select auth.uid())
        and subscriptions.status in ('active', 'trialing')
        and subscriptions.price_id = 'price_pro_monthly'
    )
  );
```

## Server-Side Subscription Check

For server components or API routes where you need to check subscription status:

```ts
// lib/subscription.ts
import { createClient } from '@/lib/supabase/server'

export async function getUserSubscription() {
  const supabase = await createClient()
  const { data: { user } } = await supabase.auth.getUser()
  if (!user) return null

  const { data: subscription } = await supabase
    .from('subscriptions')
    .select('*, prices(*, products(*))')
    .eq('user_id', user.id)
    .in('status', ['active', 'trialing'])
    .single()

  return subscription
}

export async function isProUser(): Promise<boolean> {
  const sub = await getUserSubscription()
  return sub !== null
}
```

## Passing User ID Through Checkout

When creating a Checkout Session, pass the authenticated user's ID in metadata so the webhook handler can link the Stripe Customer to your Supabase user:

```ts
// app/actions/stripe.ts
'use server'

import { stripe } from '@/lib/stripe'
import { createClient } from '@/lib/supabase/server'
import { headers } from 'next/headers'

export async function createCheckoutSession(priceId: string) {
  const supabase = await createClient()
  const { data: { user } } = await supabase.auth.getUser()
  if (!user) throw new Error('Not authenticated')

  const origin = (await headers()).get('origin')

  // Check if user already has a Stripe customer
  const { data: profile } = await supabase
    .from('users')
    .select('stripe_customer_id')
    .eq('id', user.id)
    .single()

  const sessionParams: any = {
    line_items: [{ price: priceId, quantity: 1 }],
    mode: 'subscription',
    success_url: `${origin}/success?session_id={CHECKOUT_SESSION_ID}`,
    cancel_url: `${origin}/pricing`,
    metadata: { user_id: user.id },
  }

  // Attach existing customer or set email for new customer creation
  if (profile?.stripe_customer_id) {
    sessionParams.customer = profile.stripe_customer_id
  } else {
    sessionParams.customer_email = user.email
    sessionParams.customer_creation = 'always'
  }

  const session = await stripe.checkout.sessions.create(sessionParams)
  return { url: session.url }
}
```

## Environment Variables for Vercel Deployment

Set these in your Vercel project settings:

| Variable                                | Environment        | Description                            |
|-----------------------------------------|--------------------|----------------------------------------|
| `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY`    | All                | Client-safe publishable key            |
| `STRIPE_SECRET_KEY`                     | All (server-only)  | Secret API key                         |
| `STRIPE_WEBHOOK_SECRET`                 | Production         | Production webhook signing secret      |
| `NEXT_PUBLIC_SUPABASE_URL`              | All                | Supabase project URL                   |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY`         | All                | Supabase anon/publishable key          |
| `SUPABASE_SERVICE_ROLE_KEY`             | All (server-only)  | Supabase service role key (for webhooks)|

**Important:** Use different Stripe keys and webhook secrets for test vs. production. The webhook endpoint URL in Stripe Dashboard must point to your Vercel production URL (e.g., `https://yourapp.vercel.app/api/stripe/webhook`).

## Complete Flow Checklist

1. **Supabase:** Run the SQL schema above to create tables + RLS policies
2. **Stripe Dashboard:** Create Products and Prices (or use CLI fixtures)
3. **Code:** Set up `lib/stripe.ts` server client and `lib/stripe-client.ts` browser client
4. **Code:** Create checkout Server Action with user_id in metadata
5. **Code:** Create webhook Route Handler at `app/api/stripe/webhook/route.ts`
6. **Local test:** Run `stripe listen --forward-to localhost:3000/api/stripe/webhook`
7. **Local test:** Complete a test checkout with card `4242 4242 4242 4242`
8. **Verify:** Check that subscription appears in your Supabase `subscriptions` table
9. **Vercel:** Add all environment variables
10. **Stripe Dashboard:** Create production webhook endpoint pointing to your Vercel URL
11. **Verify:** Test a real checkout on the deployed app