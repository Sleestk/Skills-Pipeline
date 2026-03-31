# Redirecting

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/redirecting

| API | Use case | Where | Status |
|---|---|---|---|
| `redirect()` | After mutation/event | Server Components, Server Functions, Route Handlers | 307 / 303 |
| `permanentRedirect()` | After canonical URL change | Server Components, Server Functions, Route Handlers | 308 |
| `useRouter().push()` | Client-side navigation | Event handlers in Client Components | N/A |
| `redirects` in `next.config.js` | Path-based before rendering | Config file | 307 / 308 |
| `NextResponse.redirect` | Condition-based (auth, etc.) | Proxy | Any |

## `redirect()`
```ts
'use server'
import { redirect } from 'next/navigation'
export async function createPost(id: string) {
  // ...mutate...
  revalidatePath('/posts')
  redirect(`/post/${id}`)  // throws — call OUTSIDE try/catch
}
```
- Returns 307 by default; 303 in Server Actions
- Accepts absolute URLs
- Can be called in Client Components during render, NOT in event handlers

## `permanentRedirect()`
```ts
import { permanentRedirect } from 'next/navigation'
permanentRedirect(`/profile/${username}`)  // 308
```
Use after canonical URL changes (e.g. username update).

## `useRouter()` (Client Components, event handlers)
```tsx
'use client'
import { useRouter } from 'next/navigation'
export default function Page() {
  const router = useRouter()
  return <button onClick={() => router.push('/dashboard')}>Dashboard</button>
}
```

## `redirects` in `next.config.js`
```ts
// next.config.ts
async redirects() {
  return [
    { source: '/about', destination: '/', permanent: true },
    { source: '/blog/:slug', destination: '/news/:slug', permanent: true },
  ]
}
```
Supports path, header, cookie, and query matching. Runs BEFORE Proxy.

## `NextResponse.redirect` in Proxy
```ts
// proxy.ts
import { NextResponse, NextRequest } from 'next/server'
import { authenticate } from 'auth-provider'

export function proxy(request: NextRequest) {
  if (!authenticate(request)) {
    return NextResponse.redirect(new URL('/login', request.url))
  }
  return NextResponse.next()
}
export const config = { matcher: '/dashboard/:path*' }
```

## Large-scale Redirects (1000+)

Store in a key-value store (Edge Config, Redis) and check in Proxy. Use a **Bloom filter** to avoid expensive lookups for non-matching paths:

1. Generate bloom filter from redirect map at build time
2. In Proxy: check bloom filter first (cheap)
3. If hit: call Route Handler to get actual redirect destination
4. Route Handler reads from redirect JSON file and returns destination

This avoids loading the entire redirect map into Proxy on every request.