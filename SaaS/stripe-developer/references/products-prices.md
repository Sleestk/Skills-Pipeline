# Products & Prices

Stripe uses a two-layer model: **Products** describe what you sell, **Prices** describe how you charge for it. A single Product can have multiple Prices (e.g., monthly and annual billing).

## Core Concepts

- **Product:** A thing you sell (e.g., "Pro Plan", "Starter Plan").
- **Price:** How much and how often (e.g., $20/month, $200/year).
- **Price ID:** The string you reference in code (e.g., `price_1PxxxxXXXX`). Always use Price IDs — never hardcode dollar amounts.

## Creating Products & Prices

### Via Stripe Dashboard (Recommended for getting started)

1. Go to [Product Catalog](https://dashboard.stripe.com/products) → **Add Product**
2. Fill in name, description
3. Add a Price: set amount, currency, billing interval (one-time, monthly, yearly)
4. Save → copy the **Price ID** (starts with `price_`)

### Via API

```ts
import { stripe } from '@/lib/stripe'

// Create a product
const product = await stripe.products.create({
  name: 'Pro Plan',
  description: 'Full access to all features',
})

// Create a recurring price for the product
const price = await stripe.prices.create({
  product: product.id,
  unit_amount: 2000, // $20.00 in cents
  currency: 'usd',
  recurring: { interval: 'month' },
})

// Create a one-time price
const oneTimePrice = await stripe.prices.create({
  product: product.id,
  unit_amount: 5000, // $50.00
  currency: 'usd',
})
```

### Via Stripe CLI Fixtures (for bootstrapping test data)

```json
// fixtures/stripe-fixtures.json
{
  "_meta": { "template_version": "0" },
  "fixtures": [
    {
      "name": "starter_product",
      "path": "/v1/products",
      "method": "post",
      "params": {
        "name": "Starter",
        "description": "Basic features for individuals"
      }
    },
    {
      "name": "starter_price",
      "path": "/v1/prices",
      "method": "post",
      "params": {
        "product": "${starter_product:id}",
        "unit_amount": 0,
        "currency": "usd",
        "recurring": { "interval": "month" }
      }
    },
    {
      "name": "pro_product",
      "path": "/v1/products",
      "method": "post",
      "params": {
        "name": "Pro",
        "description": "Full access to all features"
      }
    },
    {
      "name": "pro_price",
      "path": "/v1/prices",
      "method": "post",
      "params": {
        "product": "${pro_product:id}",
        "unit_amount": 2000,
        "currency": "usd",
        "recurring": { "interval": "month" }
      }
    }
  ]
}
```

Run: `stripe fixtures fixtures/stripe-fixtures.json`

## Pricing Models

### Flat-Rate (Fixed Price)
One price per plan, per interval. The most common SaaS model.

```
Free: $0/mo   |   Pro: $20/mo   |   Enterprise: $99/mo
```

### Freemium Model
Free tier with limited features. Pro tier unlocks everything. This is the most battle-tested SaaS pricing strategy because it lowers the barrier to entry.

Implementation: Create a free product/price ($0) and a paid product/price. Gate features in your app based on the user's subscription status stored in your database.

### Per-Seat Pricing
Charge based on the number of users/seats.

```ts
const session = await stripe.checkout.sessions.create({
  line_items: [{
    price: 'price_xxx',
    quantity: 5, // 5 seats
  }],
  mode: 'subscription',
  // ...
})
```

### Metered / Usage-Based Pricing
Charge based on actual usage (API calls, storage, etc.).

```ts
const price = await stripe.prices.create({
  product: product.id,
  currency: 'usd',
  recurring: {
    interval: 'month',
    usage_type: 'metered',
  },
  unit_amount: 10, // $0.10 per unit
})
```

Report usage with:
```ts
await stripe.subscriptionItems.createUsageRecord(subscriptionItemId, {
  quantity: 100,
  timestamp: Math.floor(Date.now() / 1000),
  action: 'increment',
})
```

### Tiered Pricing
Price changes based on volume.

```ts
const price = await stripe.prices.create({
  product: product.id,
  currency: 'usd',
  recurring: { interval: 'month' },
  billing_scheme: 'tiered',
  tiers_mode: 'graduated', // or 'volume'
  tiers: [
    { up_to: 100, unit_amount: 50 },    // First 100: $0.50 each
    { up_to: 1000, unit_amount: 30 },   // 101-1000: $0.30 each
    { up_to: 'inf', unit_amount: 10 },  // 1001+: $0.10 each
  ],
})
```

## Listing Products & Prices

```ts
// Fetch all active products with their prices
const products = await stripe.products.list({
  active: true,
  expand: ['data.default_price'],
})

// Fetch prices for a specific product
const prices = await stripe.prices.list({
  product: 'prod_xxx',
  active: true,
})
```

## Price Object Key Fields

| Field             | Description                                              |
|-------------------|----------------------------------------------------------|
| `id`              | Price ID (e.g., `price_1Pxxx`)                           |
| `product`         | Product ID this price belongs to                         |
| `unit_amount`     | Amount in smallest currency unit (cents for USD)         |
| `currency`        | Three-letter ISO currency code                           |
| `recurring`       | `{ interval, interval_count, usage_type, trial_period_days }` |
| `type`            | `one_time` or `recurring`                                |
| `active`          | Whether price can be used for new purchases              |
| `metadata`        | Custom key-value pairs                                   |

## Supabase Sync Pattern

Store products and prices in your Supabase database via webhooks. Listen for `product.created`, `product.updated`, `price.created`, `price.updated` events and sync to local tables. See `references/nextjs-supabase-integration.md` for the full schema and webhook handler.