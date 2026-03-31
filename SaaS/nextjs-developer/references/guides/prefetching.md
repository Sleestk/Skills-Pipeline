# Prefetching

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/prefetching

## Automatic Prefetch (default)

```tsx
import Link from 'next/link'
export default function NavLink() {
  return <Link href="/about">About</Link>
}
```

| Context | Prefetched payload | Client Cache TTL |
|---|---|---|
| No `loading.js` | Entire page | Until app reload |
| With `loading.js` | Layout to first loading boundary | 30s (configurable via `staleTimes`) |

Auto-prefetching only runs in **production**. Disable with `prefetch={false}`.

## Static vs Dynamic Routes

| | Static | Dynamic |
|---|---|---|
| Prefetched | Yes, full route | Only `loading.js` shell |
| Client Cache TTL | 5 min | Off (unless `staleTimes`) |
| Server roundtrip on click | No | Yes, streamed |

## Manual Prefetch

```tsx
'use client'
import { useRouter } from 'next/navigation'

export function PricingCard() {
  const router = useRouter()
  return (
    <div onMouseEnter={() => router.prefetch('/pricing')}>
      <a href="/pricing">View Pricing</a>
    </div>
  )
}
```

## Hover-triggered Prefetch

```tsx
'use client'
import Link from 'next/link'
import { useState } from 'react'

export function HoverPrefetchLink({ href, children }) {
  const [active, setActive] = useState(false)
  return (
    <Link href={href} prefetch={active ? null : false} onMouseEnter={() => setActive(true)}>
      {children}
    </Link>
  )
}
// prefetch={null} restores default (static) prefetching once user shows intent
```

## Disable Prefetch

```tsx
<Link prefetch={false} href={`/blog/${post.id}`}>{post.title}</Link>
```

## Using `useRouter` to Recreate Link Behavior

```tsx
'use client'
import { useRouter } from 'next/navigation'
import { useEffect } from 'react'

function ManualPrefetchLink({ href, children }) {
  const router = useRouter()
  useEffect(() => {
    let cancelled = false
    const poll = () => { if (!cancelled) router.prefetch(href, { onInvalidate: poll }) }
    poll()
    return () => { cancelled = true }
  }, [href, router])

  return (
    <a href={href} onClick={(e) => { e.preventDefault(); router.push(href) }}>
      {children}
    </a>
  )
}
// onInvalidate re-runs prefetch when cached data is suspected stale
```

## Troubleshooting: Side Effects During Prefetch

Server Components/layouts shouldn't have side effects (analytics, DB writes) at the top level — they run during prefetch, not just on visit. Move side effects to `useEffect` or Server Actions triggered from Client Components.

## Prefetch Scheduling

Next.js queues prefetches by priority:
1. Links in viewport
2. Links with user intent (hover/touch)
3. Newer links replace older ones
4. Links scrolled off-screen are discarded

## PPR + Prefetching

With PPR enabled, static shell is prefetched and streams immediately. Data invalidations via `revalidateTag`/`revalidatePath` silently refresh associated prefetches.