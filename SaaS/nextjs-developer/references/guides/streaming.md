# Streaming

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/streaming

## How It Works

Next.js sends the **static shell** (layouts, Suspense fallbacks) immediately, then streams dynamic content as it resolves. The browser renders progressively without waiting for all data.

**Two streams on initial load:**
1. **HTML stream** — progressive HTML chunks. When an async Server Component resolves, React streams completed HTML + inline `<script>` tags to swap fallback DOM nodes.
2. **Component payload** — serialized RSC data for hydration. Embedded in HTML on initial load; fetched alone (RSC request) on client-side navigation.

**Static shell** = everything that renders before async work. Includes layouts, navigation, Suspense fallbacks. Served from CDN with Cache Components.

## Page-Level Streaming with `loading.js`

```tsx
// app/dashboard/loading.tsx
export default function Loading() {
  return <div className="animate-pulse"><div className="h-8 bg-gray-200 rounded" /></div>
}
```

Automatically wraps `page.js` in `<Suspense>` with this fallback. Layout renders immediately; loading skeleton shown instantly; page content streams in.

> If a layout accesses uncached/runtime data (`cookies()`, `headers()`, uncached `fetch`), it blocks navigation — `loading.js` WON'T cover it. Wrap the dynamic access in its own `<Suspense>` instead.

## Granular Streaming with `<Suspense>`

### Parallel boundaries (resolve independently)
```tsx
import { Suspense } from 'react'
export default function Dashboard() {
  return (
    <div>
      <h1>Dashboard</h1>
      <Suspense fallback={<p>Loading revenue...</p>}><Revenue /></Suspense>
      <Suspense fallback={<p>Loading orders...</p>}><RecentOrders /></Suspense>
      <Suspense fallback={<p>Loading recommendations...</p>}><Recommendations /></Suspense>
    </div>
  )
}
```

### Push dynamic access down — don't await at layout level
```tsx
// Don't: const cookieStore = await cookies() at top of layout
// Do: pass the promise down
export default function DashboardLayout({ children }) {
  const cookieStore = cookies()  // start work, don't await
  return (
    <div>
      <Nav>
        <Suspense fallback={<p>Loading user...</p>}>
          <UserMenu cookiePromise={cookieStore} />
        </Suspense>
      </Nav>
      {children}
    </div>
  )
}
```

Same for `params` — pass the promise, let the consuming component await inside `<Suspense>`:
```tsx
export default function ShopPage({ params }) {
  return (
    <>
      <Hero />  {/* static shell */}
      <Suspense fallback={<p>Loading products...</p>}>
        <ProductGrid paramsPromise={params} />
      </Suspense>
    </>
  )
}
```

## Streaming Data to Client Components

Pass unawaited promise from Server Component to Client Component:
```tsx
// Server Component
const statsPromise = getStats()  // don't await
return (
  <Suspense fallback={<p>Loading chart...</p>}>
    <StatsChart dataPromise={statsPromise} />
  </Suspense>
)

// Client Component
'use client'
import { use } from 'react'
export function StatsChart({ dataPromise }) {
  const stats = use(dataPromise)  // suspends here
  return <div>...</div>
}
```

## Streaming in Route Handlers

```ts
export async function GET() {
  const encoder = new TextEncoder()
  const stream = new ReadableStream({
    async start(controller) {
      for (let i = 0; i < 10; i++) {
        controller.enqueue(encoder.encode(`Chunk ${i + 1}\n`))
        await new Promise(r => setTimeout(r, 200))
      }
      controller.close()
    },
  })
  return new Response(stream, { headers: { 'Content-Type': 'text/plain; charset=utf-8' } })
}
```

## HTTP Contract Implications

Once streaming starts, HTTP status code (200 OK) is already sent — **you cannot change it**. So:
- `notFound()` mid-stream → injects `<meta name="robots" content="noindex">`, not a real 404
- `redirect()` mid-stream → client-side redirect, not HTTP redirect

**Call `notFound()` before any `await` or `<Suspense>` if you need a real 404.**

## Web Vitals Impact

- **TTFB + FCP**: static shell sends immediately — decoupled from data fetch time
- **LCP**: keep LCP elements outside Suspense boundaries (in static shell). Use `preload` prop on `next/image` for LCP images.
- **CLS**: design skeleton fallbacks to match final content dimensions
- **INP**: selective hydration — each `<Suspense>` boundary hydrates independently, keeps main thread responsive

## Infrastructure Checklist

Streaming requires chunked transfer encoding end-to-end:
- **Nginx**: disable buffering via `X-Accel-Buffering: no`
- **Load balancers**: must support chunked encoding / HTTP/2 streaming
- **CDNs**: check streaming support in provider docs
- **Compression (gzip/Brotli)**: may buffer chunks; verify flush behavior

**Platform support**: ✅ Node.js / Docker, ❌ Static export

## Verify Streaming Works

```js
// stream-observer.mjs
const res = await fetch('http://localhost:3000/your-page', { headers: { 'Accept-Encoding': 'identity' } })
const reader = res.body.getReader()
const decoder = new TextDecoder()
let i = 0; const start = Date.now()
while (true) {
  const { done, value } = await reader.read()
  if (done) break
  console.log(`chunk ${i++} (+${Date.now() - start}ms)`)
  console.log(decoder.decode(value))
}
```

Expected output for a page with 2 Suspense boundaries: chunk 0 (static shell) → chunk 1 (~170ms, component payload) → chunk 2 (~1000ms, first boundary) → chunk 3 (~3000ms, second boundary).