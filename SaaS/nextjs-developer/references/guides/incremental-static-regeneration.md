# Incremental Static Regeneration (ISR)

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/incremental-static-regeneration

ISR lets you update static pages without rebuilding the entire site. Reduces server load, serves prerendered pages for most requests, avoids long build times.

## Basic Example

```tsx
// app/blog/[id]/page.tsx
export const revalidate = 60 // revalidate at most every 60 seconds

export async function generateStaticParams() {
  const posts = await fetch('https://api.vercel.app/blog').then(r => r.json())
  return posts.map((post) => ({ id: String(post.id) }))
}

export default async function Page({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params
  const post = await fetch(`https://api.vercel.app/blog/${id}`).then(r => r.json())
  return <main><h1>{post.title}</h1><p>{post.content}</p></main>
}
```

**How it works**: Build → generate known pages → serve from cache → after 60s, next request gets stale page + triggers background regen → fresh page replaces old. Unknown paths (e.g. `/blog/26`) are generated on-demand.

## Time-based Revalidation

```tsx
export const revalidate = 3600 // invalidate every hour
```

Recommendation: Use high revalidation times (1hr not 1s). For real-time data, use dynamic rendering.

## On-Demand Revalidation

### `revalidatePath`
```ts
'use server'
import { revalidatePath } from 'next/cache'
export async function createPost() {
  revalidatePath('/posts')
}
```

### `revalidateTag`
```ts
// Tag fetch requests:
const data = await fetch('https://api.vercel.app/blog', { next: { tags: ['posts'] } })

// Or tag unstable_cache:
const getCachedPosts = unstable_cache(async () => db.select().from(posts), ['posts'], { revalidate: 3600, tags: ['posts'] })

// Invalidate:
'use server'
import { revalidateTag } from 'next/cache'
export async function createPost() { revalidateTag('posts') }
```

## Handling Errors
If revalidation throws, the last successfully generated data continues to be served. Next request retries.

## Debugging
```js
// next.config.js
module.exports = { logging: { fetches: { fullUrl: true } } }
```

Set `NEXT_PRIVATE_DEBUG_CACHE=1` env var to log ISR cache hits/misses.

## Caveats
- Node.js runtime only (no static export)
- If multiple `fetch` requests have different `revalidate`, lowest applies to entire route
- `fetch` with `revalidate: 0` or `no-store` → route dynamically rendered
- Proxy NOT executed for on-demand ISR requests — revalidate the exact path, not a rewritten one
- Multi-instance: default per-instance. Use a [shared custom cache handler](https://nextjs.org/docs/app/api-reference/config/next-config-js/incrementalCacheHandlerPath) to coordinate

## Platform Support
- ✅ Node.js server
- ✅ Docker container
- ❌ Static export
- Platform-specific via adapters

## `x-nextjs-cache` Response Header
- `HIT` — served from cache
- `STALE` — served from cache, revalidating in background
- `MISS` — not in cache, rendered fresh
- `REVALIDATED` — regenerated via on-demand revalidation