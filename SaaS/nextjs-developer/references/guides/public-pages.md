# Public Pages

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/public-static-pages

Public pages show the same content to every user (landing pages, product pages, blogs). They can be prerendered and reused for faster loads and lower server costs.

## Pattern: Static + Cached + Streamed

### Step 1: Static components render immediately
Components with no request-time inputs are static — prerendered at build time.

### Step 2: Cache shared data
Mark data-fetching components with `'use cache'`:
```tsx
async function ProductList() {
  'use cache'
  const products = await db.product.findMany()
  return <List items={products} />
}
```
Cache components are prerendered with the rest of the page. Route stays static (`○`).

### Step 3: Stream dynamic per-user content with Suspense
```tsx
import { Suspense } from 'react'

async function PromotionContent() {
  const promotion = await getPromotion()  // user-specific, can't be cached
  return <Promotion data={promotion} />
}

export default function Page() {
  return (
    <>
      <Suspense fallback={<PromotionSkeleton />}>
        <PromotionContent />
      </Suspense>
      <Header />          {/* static shell — renders instantly */}
      <ProductList />     {/* cache component — prerendered */}
    </>
  )
}
```

**Result**: Route becomes Partial Prerendering (◐) — static shell served from CDN instantly, dynamic sections stream from server.

## Build output indicators
- `○` Static — prerendered as static content
- `◐` Partial Prerender — static HTML with dynamic server-streamed content
- `λ` Dynamic — server rendered on demand

## Key Principles
1. **Cache what's shared** — product catalogs, blog posts, marketing content
2. **Stream what's personal** — promotions, recommendations, user-specific data
3. **Keep dynamic access inside `<Suspense>`** — so it doesn't block the static shell
4. **Never await dynamic APIs at the top of a layout** — it forces the entire layout into dynamic rendering

## Demo
- Video: https://youtu.be/F6romq71KtI
- Demo: https://cache-components-public-pages.labs.vercel.dev/
- Code: https://github.com/vercel-labs/cache-components-public-pages