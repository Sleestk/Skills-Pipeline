# Webhooks

Webhooks are how Stripe tells your application that something happened — a payment succeeded, a subscription renewed, a customer cancelled. They are essential for any SaaS because many payment events happen asynchronously.

## How Webhooks Work

1. An event occurs in Stripe (e.g., successful payment)
2. Stripe sends an HTTP POST to your webhook endpoint URL
3. Your endpoint verifies the signature, processes the event, and returns `200`
4. If your endpoint doesn't return `2xx`, Stripe retries with exponential backoff for up to 3 days

## Setting Up the Webhook Endpoint

### Next.js App Router — Route Handler

```ts
// app/api/stripe/webhook/route.ts
import { headers } from 'next/headers'
import { NextResponse } from 'next/server'
import Stripe from 'stripe'
import { stripe } from '@/lib/stripe'

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
    return NextResponse.json(
      { error: 'Invalid signature' },
      { status: 400 }
    )
  }

  try {
    switch (event.type) {
      case 'checkout.session.completed': {
        const session = event.data.object as Stripe.Checkout.Session
        // Provision access — create/update user subscription in your DB
        await handleCheckoutComplete(session)
        break
      }
      case 'invoice.paid': {
        const invoice = event.data.object as Stripe.Invoice
        // Renew access — update subscription period in your DB
        await handleInvoicePaid(invoice)
        break
      }
      case 'invoice.payment_failed': {
        const invoice = event.data.object as Stripe.Invoice
        // Notify user — send email, show banner
        await handlePaymentFailed(invoice)
        break
      }
      case 'customer.subscription.updated': {
        const subscription = event.data.object as Stripe.Subscription
        // Handle upgrade/downgrade/cancel_at_period_end
        await handleSubscriptionUpdated(subscription)
        break
      }
      case 'customer.subscription.deleted': {
        const subscription = event.data.object as Stripe.Subscription
        // Revoke access
        await handleSubscriptionDeleted(subscription)
        break
      }
      default:
        console.log(`Unhandled event type: ${event.type}`)
    }
  } catch (err) {
    console.error('Error processing webhook:', err)
    return NextResponse.json(
      { error: 'Webhook handler failed' },
      { status: 500 }
    )
  }

  return NextResponse.json({ received: true })
}
```

**Critical:** The raw body must be passed to `constructEvent()` — not a parsed JSON object. In Next.js App Router, `request.text()` gives you the raw string.

## Signature Verification

Stripe signs every webhook with the `Stripe-Signature` header. Always verify:

```ts
const event = stripe.webhooks.constructEvent(
  rawBody,        // The raw request body as a string
  signature,      // The Stripe-Signature header value
  webhookSecret   // Your STRIPE_WEBHOOK_SECRET
)
```

This confirms the event came from Stripe and wasn't tampered with. **Never skip this.**

## Essential Events for SaaS

| Event                               | When It Fires                              | Action                                      |
|-------------------------------------|--------------------------------------------|---------------------------------------------|
| `checkout.session.completed`        | User completes checkout                    | Provision access, create subscription record |
| `invoice.paid`                      | Recurring payment succeeds                 | Renew access, update period dates            |
| `invoice.payment_failed`            | Recurring payment fails                    | Notify user, degrade access after grace      |
| `customer.subscription.created`     | New subscription created                   | Initial provisioning                         |
| `customer.subscription.updated`     | Plan change, cancel_at_period_end set      | Update plan, handle pending cancel           |
| `customer.subscription.deleted`     | Subscription ended                         | Revoke access                                |
| `customer.subscription.trial_will_end` | Trial ending in 3 days                  | Notify user                                  |
| `product.created` / `product.updated` | Product changes in Stripe                | Sync to local products table                 |
| `price.created` / `price.updated`   | Price changes in Stripe                    | Sync to local prices table                   |

## Local Testing with Stripe CLI

```bash
# Start listening and forward events to your local endpoint
stripe listen --forward-to localhost:3000/api/stripe/webhook

# The CLI prints a webhook secret — use it as STRIPE_WEBHOOK_SECRET
# whsec_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# In another terminal, trigger test events:
stripe trigger checkout.session.completed
stripe trigger invoice.paid
stripe trigger customer.subscription.updated
stripe trigger customer.subscription.deleted
```

## Production Setup

1. Go to [Stripe Dashboard → Developers → Webhooks](https://dashboard.stripe.com/webhooks)
2. Click **Add endpoint**
3. Enter your production URL: `https://yourdomain.com/api/stripe/webhook`
4. Select the events you want to receive
5. Copy the **Signing Secret** (`whsec_...`) and add it to your environment variables

## Best Practices

1. **Return 200 immediately.** Do your heavy processing asynchronously or after sending the response. Stripe expects a response within 20 seconds.
2. **Handle idempotently.** Stripe may send the same event more than once. Store `event.id` and skip duplicates.
3. **Don't rely solely on checkout.session.completed.** You need `invoice.paid` for renewals. Without it, access breaks after the first billing cycle.
4. **Log everything.** Log the event type and ID for every webhook you receive. This is invaluable for debugging.
5. **Use the event object, not the API.** The webhook payload contains the full object — don't make an extra API call unless you need `expand`ed data.
6. **Handle `cancel_at_period_end` in subscription.updated.** When a user cancels, this flag is set but the subscription stays `active`. Don't revoke access prematurely.

## Webhook Event Object Shape

```ts
{
  id: 'evt_xxx',
  type: 'checkout.session.completed',
  data: {
    object: {
      // The full Stripe object that triggered the event
      // e.g., Checkout Session, Invoice, Subscription
    },
    previous_attributes: {
      // Only present on .updated events — shows what changed
    }
  },
  livemode: false,
  created: 1234567890,
}
```

## Configuring Webhook Events in Vercel

When deploying to Vercel, your webhook URL is `https://your-app.vercel.app/api/stripe/webhook`. Set `STRIPE_WEBHOOK_SECRET` in your Vercel environment variables. Note: you need a **separate webhook endpoint and secret** for production vs. test mode.