# How Revalidation Works

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/how-revalidation-works

For platform engineers and advanced users implementing custom cache handlers or debugging revalidation.

## Two Types of Revalidation

- **Time-based**: stale-while-revalidate pattern. Cached content served immediately, background regen triggered when age exceeds `cacheLife`/`revalidate`. Stale served until fresh is ready.
- **On-demand**: explicit invalidation via `revalidateTag()` or `revalidatePath()`. Next request triggers fresh render.

> Pages Router on-demand ISR (`res.revalidate()`, `x-prerender-revalidate`) uses `cacheHandler` (singular). `cacheHandlers` (plural) is for `'use cache'` directives.

## What Gets Revalidated

Both **HTML response** and **RSC payload** are regenerated together in the same cache entry. They must stay in sync — if a CDN caches them with different TTLs, users may see inconsistent content during client-side navigation.

**Cross-deployment skew**: configure [`deploymentId`](https://nextjs.org/docs/app/api-reference/config/next-config-js/deploymentId) so clients from an old deploy detect a new server and trigger a hard navigation.

## Tag System

### Explicit tags
Set by developer via `cacheTag()` inside `'use cache'` or `next: { tags: [...] }` on `fetch`. `revalidateTag('my-tag')` invalidates all entries with that tag.

### Soft tags
Auto-generated from route path, prefixed `_N_T_`. `/blog/hello` generates: `_N_T_/layout`, `_N_T_/blog/layout`, `_N_T_/blog/hello/layout`, `_N_T_/blog/hello`.

`revalidatePath('/blog/hello')` works via these soft tags. In the cache handler API, soft tags are passed to `get()` as `softTags`. Use `getExpiration()` to check if any soft tag has a revalidation timestamp newer than the cache entry's own timestamp.

## Multi-Instance Coordination

By default, `revalidateTag()` on instance A only invalidates cache on that instance.

Hooks for distributed coordination in custom cache handlers:
- **`updateTags()`**: called when `revalidateTag()` is invoked — write to shared storage (Redis, DB)
- **`refreshTags()`**: called before each request — read shared storage to sync local tag state

`refreshTags()` must catch errors — if it throws, the exception propagates as a request failure. On error, requests continue with last known local state (potentially stale content).

## Implementation Patterns

### Single instance
Default filesystem cache. No config needed.

### Multi-instance with shared cache
1. Store tag invalidation timestamps in Redis/DynamoDB/HTTP API
2. Implement `updateTags()` → write to shared service
3. Implement `refreshTags()` → read from shared service (catch errors)
4. Optionally store cache entries in shared storage (atomic writes reduce mismatch window)

### CDN integration
CDN must respect `Vary` header and `Cache-Control`. Don't cache HTML and RSC with different TTLs. See [CDN Caching](/docs/app/guides/cdn-caching).

## Graceful Degradation

- **Cache write failure**: response still served, entry lost, next request re-renders
- **Cache read failure**: return `undefined` from `get()` (cache miss signal) — route renders fresh. Do NOT throw unhandled exceptions in `get()`.
- **HTML/RSC inconsistency**: use same TTL and invalidation policy, respect `Vary` header
- **Cross-deployment skew**: configure `deploymentId`