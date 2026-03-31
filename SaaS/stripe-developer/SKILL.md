---
name: stripe-developer
description: >
  Full-stack Stripe developer skill for building payment integrations with Next.js and Supabase. Covers Checkout (hosted and embedded), Products & Prices, Subscriptions & Billing, Webhooks (signature verification, event handling), Customer Portal, the Stripe Node.js SDK, client-side Stripe.js, and the Stripe CLI. Use this skill whenever the user mentions Stripe, payments, checkout, subscriptions, billing, pricing tiers, webhooks, recurring revenue, freemium, paywall, customer portal, Stripe CLI, or any task involving accepting payments. Even if the user just says "add payments", "wire up billing", "set up subscriptions", or "make this a SaaS" — trigger this skill immediately. Also trigger for connecting Stripe to Supabase (syncing subscription status via webhooks + RLS) or deploying a Stripe-integrated app to Vercel.
---

# Stripe Developer Skill

A comprehensive skill for building payment integrations in Next.js applications with Stripe. Covers five domains: **Checkout & Payments**, **Products & Pricing**, **Subscriptions & Billing**, **Webhooks**, and **Customer Portal**.

## Architecture

This skill uses progressive disclosure with reference files:

```
stripe-developer/
├── SKILL.md                          ← You are here (orchestrator)
├── references/
│   ├── checkout.md                   ← Checkout Sessions (hosted + embedded)
│   ├── products-prices.md            ← Products, Prices, and pricing models
│   ├── subscriptions.md              ← Subscriptions, billing lifecycle, metered/tiered
│   ├── webhooks.md                   ← Webhook endpoints, signature verification, event handling
│   ├── customer-portal.md            ← Self-service subscription management
│   └── nextjs-supabase-integration.md ← Cross-stack wiring patterns
```

## Routing

When the user's request arrives, determine which domain(s) it touches, then read the appropriate reference file(s):

### Checkout & Payments
If the request involves creating a checkout session, one-time payments, payment pages, redirecting to Stripe, embedded checkout, or the Stripe payment form:
→ Read `references/checkout.md`

### Products & Pricing
If the request involves creating products, setting prices, pricing tiers, free vs. pro plans, price IDs, or configuring what you sell:
→ Read `references/products-prices.md`

### Subscriptions & Billing
If the request involves recurring payments, subscription lifecycle (create, upgrade, downgrade, cancel, renew), billing intervals, trial periods, metered billing, or subscription status management:
→ Read `references/subscriptions.md`

### Webhooks
If the request involves listening for Stripe events, webhook endpoints, signature verification, handling `checkout.session.completed`, `invoice.paid`, `customer.subscription.updated`, or any event-driven payment logic:
→ Read `references/webhooks.md`

### Customer Portal
If the request involves letting customers manage their own subscriptions, update payment methods, view invoices, cancel subscriptions, or self-service billing:
→ Read `references/customer-portal.md`

### Cross-Stack Integration (Next.js + Supabase + Vercel)
If the request involves wiring Stripe into a Next.js + Supabase stack, syncing subscription status to the database, gating features behind payment status, deploying with Stripe env vars on Vercel, or building a complete SaaS payment flow:
→ Read `references/nextjs-supabase-integration.md`

### Multiple Domains
Many requests touch multiple domains. For example, "set up subscriptions with a free and pro tier in my Next.js + Supabase app" requires **Products & Pricing** + **Subscriptions** + **Webhooks** + **Cross-Stack Integration**. Read all relevant files.

## Default Stack

- **Framework**: Next.js (App Router)
- **Server SDK**: `stripe` (Node.js)
- **Client SDK**: `@stripe/stripe-js` + `@stripe/react-stripe-js`
- **Database**: Supabase (Postgres with RLS)
- **Deployment**: Vercel
- **Language**: TypeScript

## Key Principles

1. **Never expose `STRIPE_SECRET_KEY` to the client.** Use `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY` for client-side only. The secret key lives server-side exclusively.
2. **Use Server Actions or Route Handlers** to create Checkout Sessions — never create them from the client.
3. **Always verify webhook signatures** using `stripe.webhooks.constructEvent()`. Never trust unverified webhook payloads.
4. **Store subscription status locally** in your Supabase database. Don't query the Stripe API on every request — it adds latency and hits rate limits.
5. **Use `stripe listen --forward-to`** during local development to test webhooks before deploying.
6. **Test mode first, always.** Use `pk_test_` and `sk_test_` keys. Test card: `4242 4242 4242 4242`, any future expiry, any CVC.
7. **Return 200 quickly** from webhook handlers. Do async processing after acknowledging receipt.
8. **Use Price IDs, not raw amounts** — create Products and Prices in the Stripe Dashboard or API, then reference them by ID.
9. **Handle `cancel_at_period_end`** — when a user cancels, Stripe keeps the subscription active until the period ends. Don't immediately revoke access.
10. **Idempotency** — Stripe can send the same webhook event more than once. Use `event.id` to deduplicate if needed.

## Environment Variables

```env
# .env.local
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_...   # Client-safe, exposed to browser
STRIPE_SECRET_KEY=sk_test_...                     # Server-only, never expose
STRIPE_WEBHOOK_SECRET=whsec_...                   # For webhook signature verification
```

## Quick Install

```bash
npm install stripe @stripe/stripe-js @stripe/react-stripe-js
```

## Stripe Client Setup Pattern

### Server-side (singleton)

```ts
// lib/stripe.ts
import 'server-only'
import Stripe from 'stripe'

export const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!)
```

### Client-side (singleton via lazy load)

```ts
// lib/stripe-client.ts
import { loadStripe } from '@stripe/stripe-js'

let stripePromise: ReturnType<typeof loadStripe>

export function getStripe() {
  if (!stripePromise) {
    stripePromise = loadStripe(process.env.NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY!)
  }
  return stripePromise
}
```

## Stripe CLI Quick Reference

```bash
# Install
brew install stripe/stripe-cli/stripe

# Login
stripe login

# Listen for webhooks locally (forward to your Next.js app)
stripe listen --forward-to localhost:3000/api/stripe/webhook

# Trigger a test event
stripe trigger checkout.session.completed

# Create test fixtures (products + prices)
stripe fixtures fixtures/stripe-fixtures.json
```

## Test Cards

| Scenario                        | Card Number        |
|---------------------------------|--------------------|
| Payment succeeds                | `4242424242424242`  |
| Payment requires authentication | `4000002500003155`  |
| Payment is declined             | `4000000000009995`  |

Use any future expiry date and any 3-digit CVC.