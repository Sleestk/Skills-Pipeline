# Customer Portal

Stripe's Customer Portal is a pre-built, Stripe-hosted page where customers can manage their own subscriptions, update payment methods, view invoices, and cancel plans. It eliminates the need to build a custom billing management UI.

## What the Portal Provides

- Update payment method (card, bank account)
- View and download invoices
- Upgrade or downgrade subscription plan
- Cancel subscription
- View billing history
- Update billing address

## Setup

### 1. Configure in Stripe Dashboard

Before using the portal, configure it:

1. Go to [Stripe Dashboard → Settings → Billing → Customer Portal](https://dashboard.stripe.com/settings/billing/portal)
2. Enable the features you want (update payment method, cancel subscription, switch plans, etc.)
3. Configure allowed plan changes if you want users to upgrade/downgrade
4. Set cancellation policy (immediate or at period end)
5. Add your business info and branding

**Important:** Configure separately for test mode and live mode.

### 2. Create a Portal Session (Server-Side)

```ts
// app/api/stripe/portal/route.ts
import { NextResponse } from 'next/server'
import { stripe } from '@/lib/stripe'

export async function POST(request: Request) {
  const { customerId } = await request.json()
  const origin = request.headers.get('origin')

  const session = await stripe.billingPortal.sessions.create({
    customer: customerId,
    return_url: `${origin}/account`,
  })

  return NextResponse.json({ url: session.url })
}
```

### Using a Server Action (2026 pattern)

```ts
// app/actions/stripe.ts
'use server'

import { stripe } from '@/lib/stripe'
import { headers } from 'next/headers'
import { redirect } from 'next/navigation'

export async function createPortalSession(customerId: string) {
  const origin = (await headers()).get('origin')

  const session = await stripe.billingPortal.sessions.create({
    customer: customerId,
    return_url: `${origin}/account`,
  })

  redirect(session.url)
}
```

### 3. Client-Side Button

```tsx
// components/ManageSubscriptionButton.tsx
'use client'

import { createPortalSession } from '@/app/actions/stripe'

export function ManageSubscriptionButton({
  customerId,
}: {
  customerId: string
}) {
  return (
    <form action={() => createPortalSession(customerId)}>
      <button type="submit">Manage Subscription</button>
    </form>
  )
}
```

## Portal Configuration API

You can also configure the portal programmatically:

```ts
const configuration = await stripe.billingPortal.configurations.create({
  business_profile: {
    headline: 'Manage your subscription',
  },
  features: {
    customer_update: {
      enabled: true,
      allowed_updates: ['email', 'address'],
    },
    invoice_history: { enabled: true },
    payment_method_update: { enabled: true },
    subscription_cancel: {
      enabled: true,
      mode: 'at_period_end', // or 'immediately'
      cancellation_reason: {
        enabled: true,
        options: ['too_expensive', 'missing_features', 'switched_service', 'unused', 'other'],
      },
    },
    subscription_update: {
      enabled: true,
      default_allowed_updates: ['price'],
      proration_behavior: 'create_prorations',
      products: [
        {
          product: 'prod_starter',
          prices: ['price_starter_monthly', 'price_starter_yearly'],
        },
        {
          product: 'prod_pro',
          prices: ['price_pro_monthly', 'price_pro_yearly'],
        },
      ],
    },
  },
})
```

## Portal Session Parameters

| Parameter        | Description                                          |
|------------------|------------------------------------------------------|
| `customer`       | **Required.** The Stripe Customer ID                 |
| `return_url`     | URL to redirect after portal session                 |
| `configuration`  | Specific portal configuration ID (optional)          |
| `flow_data`      | Pre-select a specific flow (cancel, update, etc.)    |
| `locale`         | Language locale for the portal                       |

## Getting the Customer ID

You need the Stripe Customer ID to create a portal session. The typical pattern is:

1. When `checkout.session.completed` fires, save `session.customer` (the Stripe Customer ID) to your user's record in Supabase
2. When the user clicks "Manage Subscription", look up their Stripe Customer ID from your database
3. Use that ID to create the portal session

```ts
// In your webhook handler for checkout.session.completed:
const session = event.data.object as Stripe.Checkout.Session

await supabase
  .from('users')
  .update({ stripe_customer_id: session.customer })
  .eq('id', userId)
```

## Limitations

- **Limited customization:** The portal has a fixed layout. For a fully custom billing UI, you'd need to build your own using the Stripe API.
- **Plan change timing:** By default, upgrades/downgrades take effect at the end of the current billing period. You can configure proration for immediate changes.
- **Separate test/live configuration:** You must configure the portal separately in test and live modes.

## Portal Events (Webhooks)

| Event                                            | Trigger                              |
|--------------------------------------------------|--------------------------------------|
| `billing_portal.configuration.created`           | Portal config created                |
| `billing_portal.configuration.updated`           | Portal config updated                |
| `billing_portal.session.created`                 | Portal session opened                |
| `customer.subscription.updated`                  | User changed plan via portal         |
| `customer.subscription.deleted`                  | User cancelled via portal            |
| `payment_method.attached` / `.detached`          | User updated payment method          |