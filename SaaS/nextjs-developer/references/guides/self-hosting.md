# Self-Hosting

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/self-hosting

## Setup

- Use a **reverse proxy** (nginx) in front of Next.js — handles malformed requests, rate limiting, payload limits
- Image Optimization works with zero config via `next start`
- Proxy works with zero config via `next start` (not supported with static export)

## Environment Variables

- Server-only by default; prefix `NEXT_PUBLIC_` to expose to browser (inlined at build time)
- Read runtime env vars safely during dynamic rendering by calling `await connection()` first
- Use a single Docker image promoted through environments — runtime vars read at request time

## Caching and ISR

Default: in-memory cache (50MB) + local disk. For multi-instance/ephemeral compute, configure a shared cache handler.

### Custom cache handler
```js
// next.config.js
module.exports = { cacheHandler: require.resolve('./cache-handler.js'), cacheMaxMemorySize: 0 }
```

```js
// cache-handler.js — minimal example (extend for Redis/S3/DynamoDB)
const cache = new Map()
module.exports = class CacheHandler {
  async get(key) { return cache.get(key) }
  async set(key, data, ctx) { cache.set(key, { value: data, lastModified: Date.now(), tags: ctx.tags }) }
  async revalidateTag(tags) {
    tags = [tags].flat()
    for (let [key, value] of cache) {
      if (value.tags.some(t => tags.includes(t))) cache.delete(key)
    }
  }
  resetRequestCache() {}
}
```

Redis example: https://github.com/vercel/next.js/tree/canary/examples/cache-handler-redis

## Multi-Server Deployments

### Server Functions encryption key
Set `NEXT_SERVER_ACTIONS_ENCRYPTION_KEY` (base64 AES key, 16/24/32 bytes) so all instances use the same key:
```bash
NEXT_SERVER_ACTIONS_ENCRYPTION_KEY=your-generated-key next build
```

### Deployment identifier (version skew protection)
```js
module.exports = { deploymentId: process.env.DEPLOYMENT_VERSION }
```
When mismatch detected, Next.js triggers full page reload to fetch consistent assets.

### Shared cache + tag coordination
Implement `refreshTags()` and `updateTags()` in custom cache handler to sync invalidations across instances via Redis or similar. See [How Revalidation Works](/docs/app/guides/how-revalidation-works).

## Streaming (nginx config)
Disable response buffering:
```js
// next.config.js
async headers() {
  return [{ source: '/:path*{/}?', headers: [{ key: 'X-Accel-Buffering', value: 'no' }] }]
}
```

## Static Assets with CDN
```js
module.exports = { assetPrefix: 'https://cdn.example.com' }
```

## Build Cache (consistent across containers)
```js
module.exports = { generateBuildId: async () => process.env.GIT_HASH }
```

## `after()` and Graceful Shutdown
Send `SIGINT`/`SIGTERM` to trigger graceful shutdown. Allow 10–30s drain period so `after()` callbacks complete before exit.

## CDN Usage
- Static + fully prerendered pages: `Cache-Control: public` → CDN-cacheable
- Dynamic pages: `Cache-Control: private` → not CDN-cached
- For PPR, see [PPR Platform Guide](/docs/app/guides/ppr-platform-guide)