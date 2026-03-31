# Checkout & Payments

Stripe Checkout is the fastest way to accept payments. It provides a pre-built, Stripe-hosted (or embedded) payment page with support for cards, Apple Pay, Google Pay, and 40+ payment methods out of the box.

## Two Modes

### Stripe-Hosted Checkout (Redirect)
User clicks a button → gets redirected to `checkout.stripe.com` → completes payment → returns to your `success_url`.

**Pros:** Zero frontend payment UI to build, PCI compliance handled entirely by Stripe, supports 40+ payment methods automatically.
**Cons:** User leaves your domain temporarily.

### Embedded Checkout (2026 recommended)
An iframe/web component renders inside your Next.js page. User stays on your domain.

**Pros:** No redirect, higher conversion, user stays on your site.
**Cons:** Slightly more setup (requires `@stripe/react-stripe-js`).

## Checkout Session Modes

| Mode           | Use Case                                      |
|----------------|-----------------------------------------------|
| `payment`      | One-time payments                             |
| `subscription` | Recurring payments (subscriptions)            |
| `setup`        | Save payment method for future use            |

## Stripe-Hosted Checkout — Next.js App Router

### 1. Create a Route Handler

```ts
// app/api/checkout/route.ts
import { NextResponse } from 'next/server'
import { stripe } from '@/lib/stripe'

export async function POST(request: Request) {
  const { priceId } = await request.json()
  const origin = request.headers.get('origin')

  const session = await stripe.checkout.sessions.create({
    line_items: [{ price: priceId, quantity: 1 }],
    mode: 'subscription', // or 'payment' for one-time
    success_url: `${origin}/success?session_id={CHECKOUT_SESSION_ID}`,
    cancel_url: `${origin}/pricing`,
  })

  return NextResponse.json({ url: session.url })
}
```

### 2. Client-side redirect

```tsx
// components/CheckoutButton.tsx
'use client'

export function CheckoutButton({ priceId }: { priceId: string }) {
  const handleCheckout = async () => {
    const res = await fetch('/api/checkout', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ priceId }),
    })
    const { url } = await res.json()
    window.location.href = url
  }

  return <button onClick={handleCheckout}>Subscribe</button>
}
```

### 3. Success page

```tsx
// app/success/page.tsx
import { stripe } from '@/lib/stripe'
import { redirect } from 'next/navigation'

export default async function SuccessPage({
  searchParams,
}: {
  searchParams: Promise<{ session_id?: string }>
}) {
  const { session_id } = await searchParams
  if (!session_id) redirect('/')

  const session = await stripe.checkout.sessions.retrieve(session_id)

  if (session.status === 'complete') {
    return (
      <div>
        <h1>Payment Successful</h1>
        <p>Confirmation sent to {session.customer_details?.email}</p>
      </div>
    )
  }

  redirect('/')
}
```

## Embedded Checkout — Server Action Pattern (2026)

### 1. Server Action to create session

```ts
// app/actions/stripe.ts
'use server'

import { stripe } from '@/lib/stripe'
import { headers } from 'next/headers'

export async function createCheckoutSession(priceId: string) {
  const origin = (await headers()).get('origin')

  const session = await stripe.checkout.sessions.create({
    ui_mode: 'embedded',
    line_items: [{ price: priceId, quantity: 1 }],
    mode: 'subscription',
    return_url: `${origin}/return?session_id={CHECKOUT_SESSION_ID}`,
  })

  return { clientSecret: session.client_secret }
}
```

### 2. Embedded Checkout component

```tsx
// components/EmbeddedCheckout.tsx
'use client'

import { useCallback } from 'react'
import { loadStripe } from '@stripe/stripe-js'
import {
  EmbeddedCheckoutProvider,
  EmbeddedCheckout,
} from '@stripe/react-stripe-js'
import { createCheckoutSession } from '@/app/actions/stripe'

const stripePromise = loadStripe(
  process.env.NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY!
)

export function CheckoutForm({ priceId }: { priceId: string }) {
  const fetchClientSecret = useCallback(async () => {
    const { clientSecret } = await createCheckoutSession(priceId)
    return clientSecret!
  }, [priceId])

  return (
    <EmbeddedCheckoutProvider
      stripe={stripePromise}
      options={{ fetchClientSecret }}
    >
      <EmbeddedCheckout />
    </EmbeddedCheckoutProvider>
  )
}
```

## Checkout Session Parameters (Common)

| Parameter                       | Description                                                    |
|---------------------------------|----------------------------------------------------------------|
| `line_items`                    | Array of `{ price, quantity }` objects                         |
| `mode`                          | `payment`, `subscription`, or `setup`                          |
| `success_url`                   | Redirect after payment (hosted mode)                           |
| `cancel_url`                    | Redirect on cancel (hosted mode)                               |
| `return_url`                    | Return after payment (embedded mode)                           |
| `ui_mode`                       | `hosted` (default) or `embedded`                               |
| `customer`                      | Existing Stripe Customer ID                                    |
| `customer_email`                | Prefill email                                                  |
| `customer_creation`             | `always` to create a Customer object                           |
| `billing_address_collection`    | `auto` or `required`                                           |
| `shipping_address_collection`   | `{ allowed_countries: ['US', 'CA'] }`                          |
| `automatic_tax`                 | `{ enabled: true }` to use Stripe Tax                          |
| `allow_promotion_codes`         | `true` to enable discount codes                                |
| `metadata`                      | Key-value pairs attached to the session                        |

## Customizing the Checkout Page

- **Branding:** Set logo, colors, and fonts in [Stripe Dashboard → Settings → Branding](https://dashboard.stripe.com/settings/branding).
- **Submit button text:** Use `submit_type`: `'pay'`, `'book'`, `'donate'`, or `'auto'`.
- **Prefill email:** Pass `customer_email` or an existing `customer` ID.
- **Promotion codes:** Set `allow_promotion_codes: true`.