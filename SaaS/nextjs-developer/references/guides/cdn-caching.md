# CDN Caching

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/cdn-caching

## Cache-Control Headers Set by Next.js

| Route type | Header |
|---|---|
| Static pages (no revalidation) | `s-maxage=31536000` |
| ISR pages | `s-maxage={revalidate}, stale-while-revalidate={expire - revalidate}` |
| Dynamic pages | `private, no-cache, no-store, max-age=0, must-revalidate` |

Static assets (`/_next/static/`) get `public,max-age=31536000,immutable` (content-hashed filenames).

Use [`assetPrefix`](https://nextjs.org/docs/app/api-reference/config/next-config-js/assetPrefix) to serve static assets from a different CDN domain.

## On-Demand Revalidation + CDN

`revalidateTag()`/`revalidatePath()` invalidate the **Next.js server cache** only. CDN continues serving its cached copy until TTL expires. To propagate:
1. Call `revalidateTag()`/`revalidatePath()`
2. Call your CDN purge API for affected keys (including both HTML and RSC variants)

## PPR-Enabled Routes

For PPR routes, static prefetch responses can be CDN-cached if the CDN:
1. Includes `_rsc` search param in the cache key
2. Respects `Cache-Control` headers

## Custom Headers That Affect CDN Caching

Next.js sets `Vary` on these request headers:
- `rsc` — RSC payload vs HTML
- `next-router-state-tree` — client router state for segment updates
- `next-router-prefetch` — whether it's a prefetch
- `next-router-segment-prefetch` — specific segment being prefetched
- `next-url` — for interception routes

Many CDNs don't support `Vary`. Next.js uses `_rsc` query param as a cache-key discriminator instead.

## What You Can Safely Ignore
- `next-router-state-tree`: if omitted, server returns full payload instead of targeted update (larger but correct)
- `next-router-segment-prefetch`: falls back to broader prefetch
- `next-url`: interception routes degrade gracefully to non-intercepted page

## What You Must Preserve
- **`rsc` header**: must be forwarded server-to-client. If stripped, breaks client-side navigation.
- **`next-router-prefetch` + `_rsc` param**: required for correct prefetch caching
- **`_rsc` query param**: must be included in cache key (don't strip query params)

## Direction: Pathname-Based Cache Keying (In Progress)

Next.js is moving toward encoding all cache variability in the URL pathname, eliminating custom header dependencies:
- `/my/page.rsc` — full page RSC payload
- `/my/page.segments/path/to/segment.segment.rsc` — segment RSC payload

CDNs would simply use pathname as cache key with no `Vary` needed.

## CDN Infrastructure Compatibility Table
See [Deploying to Platforms](/docs/app/guides/deploying-to-platforms#cdn-infrastructure-compatibility) for full table (Cloudflare Workers/KV/R2, Akamai, CloudFront, Fastly, Azure, Google Cloud).