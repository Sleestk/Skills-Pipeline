# Migrating to Cache Components

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/migrating-to-cache-components
> Requires `cacheComponents: true` in `next.config.js`.

## `dynamic = "force-dynamic"`
**Not needed** — all pages are dynamic by default. Just remove it.

## `dynamic = "force-static"`
Remove it. For uncached data access, add `'use cache'` near the data access:
```tsx
import { cacheLife } from 'next/cache'
export default async function Page() {
  'use cache'
  cacheLife('max')
  const data = await fetch('https://api.example.com/data')
  return <div>...</div>
}
```
For runtime data access (`cookies()`, `headers()`, etc.), errors will guide you to wrap in `<Suspense>`.

## `revalidate`
Replace with `cacheLife`:
```tsx
// Before
export const revalidate = 3600

// After
import { cacheLife } from 'next/cache'
export default async function Page() {
  'use cache'
  cacheLife('hours')
  return <div>...</div>
}
```

## `fetchCache`
**Not needed** — with `'use cache'`, all data fetching within a cached scope is automatically cached. Remove it.

## `runtime = 'edge'`
**Not supported** with Cache Components — requires Node.js runtime. Remove `runtime = 'edge'`. Use `proxy.ts` for edge behavior on specific routes instead.

## cacheLife profiles
| Profile | Revalidation |
|---|---|
| `'seconds'` | ~10s |
| `'minutes'` | ~1m |
| `'hours'` | ~1h |
| `'days'` | ~1d |
| `'weeks'` | ~1w |
| `'max'` | ~1 year |