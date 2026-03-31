# Caching and Revalidating (Previous Model)

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/caching-without-cache-components
> Use this guide when **not** using Cache Components (`cacheComponents` flag).

## Caching fetch Requests

`fetch` requests are NOT cached by default. Opt in with `cache: 'force-cache'`:
```tsx
const data = await fetch('https://...', { cache: 'force-cache' })
```

## `unstable_cache` for non-fetch functions

```ts
import { unstable_cache } from 'next/cache'
export const getCachedUser = unstable_cache(
  async (id: string) => db.select().from(users).where(eq(users.id, id)).then(r => r[0]),
  ['user'],           // cache key prefix
  { tags: ['user'], revalidate: 3600 }
)
```

## Route Segment Config

### `dynamic`
```tsx
export const dynamic = 'auto' // 'auto' | 'force-dynamic' | 'error' | 'force-static'
```
- `'auto'` — default, cache as much as possible
- `'force-dynamic'` — always render at request time (no caching)
- `'error'` — force prerender, error if dynamic APIs used
- `'force-static'` — force prerender, `cookies()`/`headers()`/`useSearchParams()` return empty values

### `fetchCache` (advanced)
```tsx
export const fetchCache = 'auto'
// 'auto' | 'default-cache' | 'only-cache' | 'force-cache'
// 'default-no-store' | 'only-no-store' | 'force-no-store'
```

### `revalidate`
```tsx
export const revalidate = false // false | 0 | number (seconds)
```
- `false` — cache indefinitely (stale-while-revalidate model)
- `0` — always dynamic
- `number` — revalidate every N seconds

## Time-based Revalidation

```tsx
const data = await fetch('https://...', { next: { revalidate: 3600 } })
```

## On-Demand Revalidation

### Tag cached data
```ts
const data = await fetch('https://...', { next: { tags: ['user'] } })
```

### Invalidate by tag
```ts
import { revalidateTag } from 'next/cache'
export async function updateUser(id: string) {
  // mutate...
  revalidateTag('user')
}
```

### Invalidate by path
```ts
import { revalidatePath } from 'next/cache'
revalidatePath('/profile')
```

## Deduplicating Requests (non-fetch)
```ts
import { cache } from 'react'
export const getPost = cache(async (id: string) => {
  return db.query.posts.findFirst({ where: eq(posts.id, parseInt(id)) })
})
```

## Preloading Data
```ts
// utils/get-item.ts
import { cache } from 'react'
import 'server-only'
export const getItem = cache(async (id: string) => { /* ... */ })
export const preload = (id: string) => { void getItem(id) }
```

Call `preload(id)` before blocking work so data fetches in parallel.