# Deploying to Platforms

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/deploying-to-platforms

## Minimum Requirements
A Node.js server. `next start` handles everything: Server Components, ISR, PPR, Cache Components, Server Actions, Proxy, `after()`. Additional infrastructure (CDN, edge compute, shared cache) improves performance but isn't required for correctness.

## Functional vs Performance Fidelity
- **Functional fidelity** — every feature works correctly (binary: passes adapter test suite or doesn't)
- **Performance fidelity** — features achieve optimal perf (spectrum: CDN latency for PPR shell, sub-second ISR, etc.)

## Feature Support Matrix

| Feature | Streaming | Shared Cache | Edge Stitching |
|---|---|---|---|
| Server Components | Required | No | No |
| ISR (time-based) | No | Recommended | No |
| ISR (on-demand) | No | Recommended | No |
| Partial Prerendering | Required | Recommended | Optional |
| Cache Components (`use cache`) | Required | Recommended | No |
| Proxy / Middleware | No | No | No |
| Server Actions | Required | No | No |
| `after()` | No | No | No (needs graceful shutdown) |

**Streaming Required** = must support chunked transfer encoding or HTTP/2 streaming without buffering.

**Shared Cache Recommended** = use `cacheHandler` (ISR/fetch) and `cacheHandlers` (`use cache`) for cross-instance consistency.

## CDN Infrastructure Compatibility

| CDN | Edge Compute | KV/Tags | Blob Storage | PPR Resuming |
|---|---|---|---|---|
| Cloudflare | Workers | KV | R2 | Yes (worker) |
| Akamai | EdgeWorkers | EdgeKV | Object Storage | Yes (worker) |
| AWS CloudFront | Lambda@Edge | KeyValueStore | S3 | Yes (Lambda) |
| Fastly | Compute | KV Store | Object Storage | Yes (WASM) |
| Azure | Functions | Managed Redis | Blob Storage | Yes (server) |
| Google Cloud | Cloud Run | Various KV | Cloud Storage | Yes (server) |

Most adapters today deploy as Node.js/Docker without leveraging CDN-specific primitives.

## Adapters

The [Deployment Adapter API](https://nextjs.org/docs/app/api-reference/config/next-config-js/adapterPath) lets platforms customize build output. Two runtime caching interfaces:
- `cacheHandler` (singular) — ISR, route handlers, `fetch`, `unstable_cache`, image optimization
- `cacheHandlers` (plural) — `'use cache'` directive backends

### Verified Adapters (open source + run full test suite)
- **Vercel** — https://vercel.com/docs/frameworks/nextjs
- **Bun** — https://bun.sh/docs/frameworks/nextjs

Cloudflare and Netlify building verified adapters; offer own integrations in the meantime.

### Other platforms (own integrations)
Appwrite, AWS Amplify, Cloudflare Workers, Deno Deploy, Firebase App Hosting, Netlify — feature support varies, check each provider's docs.