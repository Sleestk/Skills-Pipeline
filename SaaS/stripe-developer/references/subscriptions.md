# Subscriptions & Billing

Subscriptions are the core of SaaS billing. Stripe handles the full lifecycle: creation, renewal, upgrade, downgrade, cancellation, payment failures, and recovery.

## Subscription Lifecycle

```
Customer subscribes
    → checkout.session.completed (provision access)
    → invoice.paid (each renewal)
    → invoice.payment_failed (payment issue)
    → customer.subscription.updated (plan change, cancel_at_period_end)
    → customer.subscription.deleted (subscription ended)
```

A SaaS business lives and dies by what happens *between* payments. You must handle every stage.

## Subscription Statuses

| Status                 | Meaning                                                    |
|------------------------|------------------------------------------------------------|
| `trialing`             | In free trial period                                       |
| `active`               | Paid and current                                           |
| `past_due`             | Payment failed, retrying                                   |
| `canceled`             | Cancelled (either immediately or at period end)            |
| `incomplete`           | Initial payment failed                                     |
| `incomplete_expired`   | Initial payment failed and window expired                  |
| `unpaid`               | All retry attempts exhausted                               |
| `paused`               | Subscription paused                                        |

## Creating Subscriptions via Checkout

The simplest path — use Checkout in `subscription` mode:

```ts
const session = await stripe.checkout.sessions.create({
  line_items: [{ price: 'price_pro_monthly', quantity: 1 }],
  mode: 'subscription',
  success_url: `${origin}/success?session_id={CHECKOUT_SESSION_ID}`,
  cancel_url: `${origin}/pricing`,
})
```

This creates the Customer, Subscription, Invoice, and PaymentIntent all in one step.

## Creating Subscriptions via API

For custom flows where you already have a Customer and PaymentMethod:

```ts
const subscription = await stripe.subscriptions.create({
  customer: 'cus_xxx',
  items: [{ price: 'price_pro_monthly' }],
  payment_behavior: 'default_incomplete',
  payment_settings: {
    save_default_payment_method: 'on_subscription',
  },
  expand: ['latest_invoice.payment_intent'],
})
```

## Free Trials

```ts
const session = await stripe.checkout.sessions.create({
  line_items: [{ price: 'price_pro_monthly', quantity: 1 }],
  mode: 'subscription',
  subscription_data: {
    trial_period_days: 14,
  },
  success_url: `${origin}/success?session_id={CHECKOUT_SESSION_ID}`,
  cancel_url: `${origin}/pricing`,
})
```

Trial behavior: Stripe collects the payment method upfront but doesn't charge until the trial ends. If the trial expires without cancellation, the first invoice is generated.

## Upgrading / Downgrading

Retrieve the subscription, then update the Price:

```ts
const subscription = await stripe.subscriptions.retrieve('sub_xxx')

await stripe.subscriptions.update('sub_xxx', {
  items: [{
    id: subscription.items.data[0].id,
    price: 'price_new_plan',
  }],
  proration_behavior: 'create_prorations', // or 'none', 'always_invoice'
})
```

**Proration:** By default, Stripe calculates the difference between old and new prices and applies a credit/charge on the next invoice.

## Cancellation

### Cancel at period end (recommended)
User keeps access until the current billing period ends:

```ts
await stripe.subscriptions.update('sub_xxx', {
  cancel_at_period_end: true,
})
```

The subscription status stays `active` with `cancel_at_period_end: true`. It transitions to `canceled` when the period ends. **Do not revoke access immediately on cancel** — this is a common mistake.

### Cancel immediately

```ts
await stripe.subscriptions.cancel('sub_xxx')
```

Access should be revoked immediately.

### Reactivating a pending cancellation

If the user changes their mind before period end:

```ts
await stripe.subscriptions.update('sub_xxx', {
  cancel_at_period_end: false,
})
```

## Payment Failures & Recovery

When a recurring payment fails, Stripe enters Smart Retries — it retries the charge using ML to pick optimal retry times. You should:

1. Listen for `invoice.payment_failed` webhook events
2. Notify the user to update their payment method
3. Send them to the Customer Portal (see `references/customer-portal.md`)

Configure retry behavior in [Stripe Dashboard → Settings → Billing → Subscriptions](https://dashboard.stripe.com/settings/billing/automatic).

## Retrieving Subscription Data

```ts
// Get a specific subscription
const sub = await stripe.subscriptions.retrieve('sub_xxx', {
  expand: ['default_payment_method', 'latest_invoice'],
})

// List all subscriptions for a customer
const subs = await stripe.subscriptions.list({
  customer: 'cus_xxx',
  status: 'all',
})
```

## Key Subscription Object Fields

| Field                    | Description                                           |
|--------------------------|-------------------------------------------------------|
| `id`                     | Subscription ID (`sub_xxx`)                           |
| `customer`               | Customer ID                                           |
| `status`                 | Current status (see table above)                      |
| `items.data`             | Array of subscription items (price + quantity)         |
| `current_period_start`   | Start of current billing period (Unix timestamp)      |
| `current_period_end`     | End of current billing period (Unix timestamp)        |
| `cancel_at_period_end`   | Whether cancellation is pending                       |
| `canceled_at`            | When the sub was canceled                             |
| `trial_start`            | Trial start timestamp                                 |
| `trial_end`              | Trial end timestamp                                   |
| `default_payment_method` | Payment method ID                                     |
| `latest_invoice`         | Most recent invoice ID                                |
| `metadata`               | Custom key-value pairs                                |

## Common Mistakes

1. **Only listening to `checkout.session.completed`** — You also need `invoice.paid` for renewals. Without it, users lose access after the first billing cycle.
2. **Storing subscription data only in Stripe** — Your app needs a local copy. Querying Stripe on every request adds latency and hits rate limits.
3. **Immediately revoking access on cancel** — `cancel_at_period_end` keeps the sub active until period end. Respect it.
4. **Forgetting to test with Stripe CLI** — Run `stripe listen --forward-to localhost:3000/api/stripe/webhook` locally.
5. **Not handling proration on plan changes** — Decide on `proration_behavior` before users can upgrade/downgrade.