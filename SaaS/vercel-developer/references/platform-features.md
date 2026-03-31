# Platform Features

Vercel provides infrastructure features beyond deployment: serverless/edge functions, storage, caching, analytics, and integrations with the SaaS stack.

## Serverless Functions

Every Next.js API route (Route Handler) and Server Action automatically becomes a Serverless Function on Vercel. No configuration needed.

**Default behavior:**
- Runs in a single region (configurable)
- Cold starts on first invocation, then warm for subsequent requests
- 10-second timeout (Hobby), 60-second (Pro), 900-second (Enterprise)
- Max 250MB function size (compressed)

**Configuring region:**
```json
// vercel.json
{
  "regions": ["iad1"]  // US East (Washington, D.C.)
}
```

Or per-function via export:
```ts
// app/api/heavy/route.ts
export const runtime = 'nodejs' // default
export const maxDuration = 30   // seconds (Pro plan)
export const preferredRegion = 'iad1'
```

## Edge Functions

Edge Functions run globally on Vercel's edge network — low latency, close to users. Next.js Middleware (now Proxy in v16) runs on the edge by default.

```ts
// app/api/fast/route.ts
export const runtime = 'edge'

export async function GET() {
  return new Response('Hello from the edge!')
}
```

**Edge limitations:** No Node.js APIs (fs, child_process, etc.), limited to Web APIs, smaller bundle size limit, shorter timeout.

**Use Edge for:** Auth checks, redirects, A/B testing, geolocation, simple transformations.
**Use Serverless for:** Database queries, heavy computation, file processing, anything needing Node.js APIs.

## Caching & ISR

### Static Generation
Pages without dynamic data are statically generated at build time and served from CDN.

### ISR (Incremental Static Regeneration)
Regenerate static pages in the background after a specified interval:

```tsx
// app/blog/[slug]/page.tsx
export const revalidate = 60 // Revalidate every 60 seconds

export default async function BlogPost({ params }) {
  const post = await getPost((await params).slug)
  return <article>{post.content}</article>
}
```

### On-Demand Revalidation
Revalidate specific pages or cache tags when data changes:

```ts
// app/api/revalidate/route.ts
import { revalidatePath, revalidateTag } from 'next/cache'

export async function POST(request: Request) {
  const { path, tag } = await request.json()

  if (path) revalidatePath(path)
  if (tag) revalidateTag(tag)

  return Response.json({ revalidated: true })
}
```

### CDN Cache
Vercel's CDN automatically caches static assets and ISR pages at the edge. Cache-Control headers are managed by Next.js automatically.

### Cache Management via CLI
```bash
vercel cache invalidate --tag blog
vercel cache purge --type cdn
vercel cache purge --type data
```

## Vercel Storage

### Vercel Postgres
Serverless PostgreSQL database. Good for small-to-medium apps. For the SaaS stack in the script, **use Supabase instead** — it provides Postgres + Auth + RLS out of the box.

### Vercel KV
Redis-compatible key-value store (powered by Upstash). Good for:
- Session storage
- Rate limiting
- Feature flags
- Short-lived cache

```ts
import { kv } from '@vercel/kv'

await kv.set('user:123:plan', 'pro')
const plan = await kv.get('user:123:plan')
```

### Vercel Blob
File storage for uploads, images, PDFs, etc.

```ts
import { put, del, list } from '@vercel/blob'

// Upload a file
const blob = await put('avatars/user-123.jpg', file, {
  access: 'public',
})

// List files
const { blobs } = await list({ prefix: 'avatars/' })

// Delete
await del(blob.url)
```

## Analytics & Monitoring

### Vercel Analytics
Real user monitoring (RUM) — tracks Core Web Vitals (LCP, FID, CLS) from actual visitors.

```bash
npm install @vercel/analytics
```

```tsx
// app/layout.tsx
import { Analytics } from '@vercel/analytics/react'

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        {children}
        <Analytics />
      </body>
    </html>
  )
}
```

### Vercel Speed Insights
Lab-based performance scores for each deployment.

```bash
npm install @vercel/speed-insights
```

```tsx
import { SpeedInsights } from '@vercel/speed-insights/next'

// Add to layout.tsx alongside Analytics
<SpeedInsights />
```

## Integrations

Vercel has a marketplace of integrations. Key ones for the SaaS stack:

### Supabase Integration
Automatically sets `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`, and `SUPABASE_SERVICE_ROLE_KEY` in your Vercel project.

```bash
vercel integration add supabase
```

Or install via [Vercel Marketplace → Supabase](https://vercel.com/integrations/supabase).

### Other Useful Integrations
- **Stripe** — No official integration needed; just set env vars manually
- **Sentry** — Error tracking and monitoring
- **Axiom** — Log management
- **Upstash** — Redis/KV and QStash (message queue)

## Web Application Firewall (WAF)

Available on Pro/Enterprise. Protects against DDoS, bot attacks, and malicious traffic. Configure rules in Project Settings → Firewall.

## Draft Mode

Preview draft content from a headless CMS without publishing:

```ts
// app/api/draft/route.ts
import { draftMode } from 'next/headers'

export async function GET() {
  (await draftMode()).enable()
  return new Response('Draft mode enabled')
}
```

On Vercel, Draft Mode is secured behind authentication automatically.

## Partial Prerendering (PPR) — Experimental

Combine static and dynamic content in a single page. The static shell is served immediately from CDN, while dynamic holes stream in.

```tsx
import { Suspense } from 'react'

export default function Dashboard() {
  return (
    <main>
      <h1>Dashboard</h1> {/* Static — served from CDN */}
      <Suspense fallback={<p>Loading...</p>}>
        <DynamicStats /> {/* Dynamic — streamed in */}
      </Suspense>
    </main>
  )
}
```

Enable in `next.config.ts`:
```ts
const nextConfig = {
  experimental: {
    ppr: true,
  },
}
```

## Vercel + Next.js Specific Features

| Feature                | What It Does                                           |
|------------------------|--------------------------------------------------------|
| Image Optimization     | `next/image` auto-optimizes images on Vercel's edge    |
| Font Optimization      | `next/font` self-hosts fonts from Google/custom fonts  |
| Middleware on Edge      | `proxy.ts` runs globally on every request              |
| Streaming              | Server Components stream chunks as they resolve        |
| Server Actions         | Run server code from client forms/buttons              |

## Free Tier Limits (Hobby Plan)

| Resource                | Limit                    |
|-------------------------|--------------------------|
| Bandwidth               | 100 GB / month           |
| Serverless Function Invocations | 100,000 / month |
| Builds                  | Unlimited                |
| Preview Deployments     | Unlimited                |
| Serverless Timeout      | 10 seconds               |
| Team members            | 1                        |
| Commercial use          | Not allowed              |

For production SaaS, you'll need the **Pro plan** ($20/month per member) which includes commercial use, 60-second function timeout, 1TB bandwidth, and more.