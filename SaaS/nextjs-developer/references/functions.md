# Next.js Functions Reference (v16.2.1)

## Server-Side APIs (from `next/headers` / `next/navigation` / `next/server`)

### `cookies()` — async, from `next/headers`
Read-only in Server Components. Read/write in Server Actions and Route Handlers.
```tsx
const cookieStore = await cookies()
cookieStore.get('name')          // → { name, value } | undefined
cookieStore.getAll()             // → array of all cookies
cookieStore.has('name')          // → boolean
cookieStore.set('name', 'value', { secure: true, httpOnly: true, maxAge: 3600 })
cookieStore.delete('name')
```
- Cannot `set`/`delete` during Server Component rendering — only in Server Actions/Route Handlers
- Opts route into dynamic rendering (Request-time API)

### `headers()` — async, from `next/headers`
Read-only. Returns Web `Headers` object.
```tsx
const headersList = await headers()
headersList.get('user-agent')
headersList.has('authorization')
```
- Opts route into dynamic rendering

### `draftMode()` — async, from `next/headers`
```tsx
const { isEnabled } = await draftMode()
// In Route Handler:
const draft = await draftMode()
draft.enable()   // sets __prerender_bypass cookie
draft.disable()
```

### `redirect(path, type?)` — from `next/navigation`
Throws `NEXT_REDIRECT`. Call **outside** try/catch blocks.
- Returns 307 (temporary). Use `permanentRedirect()` for 308.
- In Server Actions: defaults to `push`. Elsewhere: defaults to `replace`.
- Works in Server Components, Client Components (render only, not event handlers), Server Actions, Route Handlers.
```tsx
import { redirect } from 'next/navigation'
if (!team) redirect('/login')
```

### `permanentRedirect(path, type?)` — from `next/navigation`
Same as `redirect` but returns 308 (permanent).

### `notFound()` — from `next/navigation`
Throws `NEXT_HTTP_ERROR_FALLBACK;404`. Renders `not-found.tsx` and injects `noindex` meta.
```tsx
import { notFound } from 'next/navigation'
if (!user) notFound()
```

### `forbidden()` — from `next/navigation` (experimental)
Requires `experimental.authInterrupts: true` in `next.config.ts`. Returns 403, renders `forbidden.tsx`.
```tsx
import { forbidden } from 'next/navigation'
if (session.role !== 'admin') forbidden()
```

### `unauthorized()` — from `next/navigation` (experimental)
Same pattern — requires `authInterrupts`. Returns 401, renders `unauthorized.tsx`.
```tsx
import { unauthorized } from 'next/navigation'
if (!session) unauthorized()
```

### `connection()` — from `next/server`
Defers rendering to request time without using a Request-time API. Replaces deprecated `unstable_noStore`.
```tsx
import { connection } from 'next/server'
await connection()
// Everything below excluded from prerendering
const rand = Math.random()
```

### `after(callback)` — from `next/server`
Runs callback **after** response is sent. For logging/analytics that shouldn't block the response.
```tsx
import { after } from 'next/server'
after(() => log())
```
- Works in Server Components, Route Handlers, Server Actions, Proxy
- In Server Components: read `cookies()`/`headers()` **outside** the `after` callback, then pass values in
- In Route Handlers/Server Actions: can call `cookies()`/`headers()` inside the callback directly

---

## Cache Invalidation (from `next/cache`)

### `revalidatePath(path, type?)`
Invalidates a specific path on next visit. Call in Server Actions or Route Handlers.
```tsx
revalidatePath('/blog')                     // specific page
revalidatePath('/blog/[slug]', 'page')      // all matching pages
revalidatePath('/blog/[slug]', 'layout')    // layout + all nested pages
revalidatePath('/', 'layout')               // purge everything
```
- Use **destination** path when rewrites are configured, not the source path

### `revalidateTag(tag, profile?)`
Marks tagged data as stale across all pages using that tag.
```tsx
// Recommended: stale-while-revalidate
revalidateTag('posts', 'max')

// Immediate expiry (for webhooks)
revalidateTag('posts', { expire: 0 })
```
- Works in Server Actions and Route Handlers
- Single-argument form `revalidateTag('tag')` is deprecated

### `updateTag(tag)` — Server Actions only
Immediately expires tagged cache. Next request waits for fresh data. For read-your-own-writes.
```tsx
// In Server Action after mutation:
updateTag('posts')
updateTag(`post-${post.id}`)
redirect(`/posts/${post.id}`)
```
- Cannot be used in Route Handlers — use `revalidateTag` there instead

### `refresh()` — Server Actions only (from `next/cache`)
Refreshes the client router without full page reload.
```tsx
import { refresh } from 'next/cache'
refresh()
```

### `cacheTag(...tags)` and `cacheLife(profile)` — inside `'use cache'` scopes
See main SKILL.md Step 2 for full docs. Quick reference:
```tsx
import { cacheTag, cacheLife } from 'next/cache'

async function getData() {
  'use cache'
  cacheTag('my-data')                       // tag for invalidation
  cacheLife('hours')                        // preset: seconds/minutes/hours/days/weeks/max
  cacheLife({ stale: 300, revalidate: 900, expire: 86400 })  // inline
  return fetch('/api/data').then(r => r.json())
}
```

---

## Static Generation

### `generateStaticParams()`
Return array of param objects to pre-generate dynamic routes at build time.
```tsx
export async function generateStaticParams() {
  const posts = await fetch('https://.../posts').then(r => r.json())
  return posts.map(post => ({ slug: post.slug }))
}
// Pair with dynamicParams = false to 404 unspecified paths
export const dynamicParams = false
```
- Works on pages, layouts, and Route Handlers
- `fetch` requests inside are automatically memoized

### `generateMetadata({ params, searchParams }, parent)`
Dynamic metadata — Server Components only. `params`/`searchParams` are Promises.
```tsx
export async function generateMetadata({ params }): Promise<Metadata> {
  const { slug } = await params
  const post = await getPost(slug)
  return { title: post.title, openGraph: { images: [post.image] } }
}
```
Key `Metadata` fields: `title`, `description`, `openGraph`, `twitter`, `robots`, `icons`, `metadataBase`, `alternates`, `verification`, `manifest`

`title` template pattern:
```tsx
// In layout.tsx
export const metadata = { title: { template: '%s | My Site', default: 'My Site' } }
// In page.tsx — outputs "About | My Site"
export const metadata = { title: 'About' }
```

### `generateViewport({ params })`
```tsx
export const viewport: Viewport = { themeColor: 'black', colorScheme: 'dark' }
// Or dynamic:
export function generateViewport({ params }) { return { themeColor: '...' } }
```

---

## Client Hooks (all require `'use client'`)

### `useRouter()` — from `next/navigation`
```tsx
const router = useRouter()
router.push('/dashboard')             // navigate + add history
router.replace('/dashboard')          // navigate, replace history
router.refresh()                      // re-fetch Server Components, clear client cache
router.prefetch('/dashboard')         // manual prefetch
router.back() / router.forward()
```
- Import from `next/navigation`, NOT `next/router`
- Prefer `<Link>` for navigation when possible

### `usePathname()` — from `next/navigation`
```tsx
const pathname = usePathname()  // '/dashboard'
```
- Wrap in `<Suspense>` when route has dynamic params with `cacheComponents` enabled

### `useSearchParams()` — from `next/navigation`
```tsx
const searchParams = useSearchParams()  // URLSearchParams (read-only)
searchParams.get('q')       // '...' | null
searchParams.has('q')       // boolean
searchParams.getAll('tag')  // string[]
```
- Wrap in `<Suspense>` to avoid blocking prerendering

### `useParams()` — from `next/navigation`
```tsx
// Route: /shop/[tag]/[item], URL: /shop/shoes/nike
const params = useParams()  // { tag: 'shoes', item: 'nike' }
```

### `useSelectedLayoutSegment(parallelRoutesKey?)` — from `next/navigation`
Returns active segment **one level below** the layout where called.
```tsx
const segment = useSelectedLayoutSegment()  // 'dashboard' | null
```

### `useSelectedLayoutSegments(parallelRoutesKey?)` — from `next/navigation`
Returns all active segments below the layout.
```tsx
const segments = useSelectedLayoutSegments()  // ['dashboard', 'settings']
```

### `useLinkStatus()` — from `next/link` (v15.3+)
Track pending state of a `<Link>`. Must be inside a `<Link>` descendant.
```tsx
'use client'
import { useLinkStatus } from 'next/link'
function Hint() {
  const { pending } = useLinkStatus()
  return <span className={pending ? 'loading' : ''} />
}
// Use inside <Link> children only
```

### `useReportWebVitals(callback)` — from `next/web-vitals`
```tsx
import { useReportWebVitals } from 'next/web-vitals'
useReportWebVitals((metric) => {
  // metric.name: FCP | LCP | FID | CLS | INP | TTFB
  // metric.value, metric.rating: 'good' | 'needs-improvement' | 'poor'
  navigator.sendBeacon('/analytics', JSON.stringify(metric))
})
```
Create a `WebVitals` client component and import it into the root layout.

---

## Error Handling

### `unstable_rethrow(err)` — from `next/navigation`
Re-throw Next.js internal errors (from `notFound()`, `redirect()`, etc.) that were accidentally caught.
```tsx
try {
  const data = await fetch(url).then(r => {
    if (r.status === 404) notFound()
    return r.json()
  })
} catch (err) {
  unstable_rethrow(err)  // re-throws Next.js errors, lets app errors fall through
  console.error(err)
}
```

### `unstable_catchError(fallback)` — from `next/error` (v16.2+)
Programmatic error boundary alternative to `error.tsx`. Works anywhere in component tree.
```tsx
'use client'
import { unstable_catchError, type ErrorInfo } from 'next/error'

function ErrorFallback(props: { title: string }, { error, unstable_retry }: ErrorInfo) {
  return (
    <div>
      <h2>{props.title}</h2>
      <p>{error.message}</p>
      <button onClick={() => unstable_retry()}>Try again</button>
    </div>
  )
}
export default unstable_catchError(ErrorFallback)

// Usage:
// <ErrorWrapper title="Dashboard Error">{children}</ErrorWrapper>
```

---

## Image Generation

### `ImageResponse` — from `next/og`
Generate dynamic images (OG, Twitter cards) using JSX.
```tsx
import { ImageResponse } from 'next/og'

export default function Image() {
  return new ImageResponse(
    <div style={{ display: 'flex', fontSize: 128, background: 'white', width: '100%', height: '100%' }}>
      Hello World
    </div>,
    { width: 1200, height: 630 }
  )
}
```
- Use in `opengraph-image.tsx`, `twitter-image.tsx`, or Route Handlers
- Only flexbox CSS supported (no grid)
- Max bundle 500KB; supports `.ttf`/`.otf`/`.woff` fonts

---

## Proxy / Request Utilities (from `next/server`)

### `NextRequest` (extends `Request`)
```tsx
request.cookies.get('name')          // cookie access
request.nextUrl.pathname             // '/dashboard'
request.nextUrl.searchParams         // URLSearchParams
request.nextUrl.basePath             // base path
```

### `NextResponse` (extends `Response`)
```tsx
NextResponse.redirect(new URL('/home', request.url))
NextResponse.rewrite(new URL('/proxy-target', request.url))
NextResponse.next()                  // continue to route
NextResponse.next({ request: { headers: newHeaders } })  // forward modified headers upstream
NextResponse.json({ error: 'msg' }, { status: 500 })
```
**Important:** `NextResponse.next({ headers })` sends headers to the client — avoid this. Use `NextResponse.next({ request: { headers } })` to forward upstream instead.

### `userAgent(request)` — from `next/server`
```tsx
import { userAgent } from 'next/server'
const { device, browser, os, isBot } = userAgent(request)
// device.type: 'mobile' | 'tablet' | 'desktop' | undefined
```

---

## Legacy / Deprecated APIs

| API | Status | Replace With |
|---|---|---|
| `unstable_noStore()` | Deprecated | `connection()` |
| `unstable_cache()` | Deprecated (v16) | `'use cache'` directive |
| `export const dynamic = 'force-dynamic'` | Still works | `connection()` or Request-time APIs |
| `revalidateTag(tag)` (1 arg) | Deprecated | `revalidateTag(tag, 'max')` |
| `cookies` (sync) | Deprecated | `await cookies()` |
| `headers` (sync) | Deprecated | `await headers()` |
| `draftMode` (sync) | Deprecated | `await draftMode()` |
| `params` (sync) | Deprecated | `await params` |