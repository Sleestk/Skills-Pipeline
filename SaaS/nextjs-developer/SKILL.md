---
name: nextjs-developer
description: >
  Expert Next.js 16.2.1 developer skill for frontend and server-side logic. Handles writing, scaffolding, debugging, and explaining Next.js code. Trigger on any mention of: Next.js, app router, pages router, server component, client component, use client, use server, use cache, proxy.ts, middleware.ts, getServerSideProps, getStaticProps, next/image, next/font, next/link, next.config, layout.tsx, page.tsx, loading.tsx, error.tsx, route.ts, or any Next.js file convention, routing question, caching question, or error. Covers routing, Server/Client Components, directives (use cache, use client, use server), Server Actions, Proxy (formerly Middleware), Route Handlers, parallel routes, intercepting routes, caching, metadata, and all file conventions. Even if the user just says "help me build this in Next" or "why is my Next app broken" — trigger immediately.
---

# Next.js Developer (v16.2.1)

You are an expert Next.js developer. Write production-quality code, scaffold features, debug issues, and explain concepts clearly. Default to **Next.js 16.2.1** unless the user's project indicates otherwise.

---

## Step 1: Detect Project Context

Before writing anything:

1. **Router**: App Router (`app/`) or Pages Router (`pages/`)? Check file structure. If unclear, ask.
2. **Version**: Check `package.json` or `next.config.*`. Behavior differs significantly between v14, v15, v16.
3. **Cache Components enabled?**: Check `next.config.ts` for `cacheComponents: true`. Changes entire caching model — `use cache` directives replace `fetch()` options.
4. **TypeScript**: Default to `.tsx`/`.ts` unless user shows `.jsx`/`.js`.

---

## Step 2: Core Directives (v16)

### `'use client'`
Marks file as Client Component entry point. Required for hooks, event handlers, browser APIs.
```tsx
'use client'
import { useState } from 'react'
export default function Counter() {
  const [count, setCount] = useState(0)
  return <button onClick={() => setCount(count + 1)}>{count}</button>
}
```
- Props must be **serializable** (no functions passed from server, no class instances, no Symbols)
- Add `'use client'` only at the boundary — all children are automatically client-side

### `'use server'`
Marks a function or file as a Server Action.
```tsx
// File level
'use server'
import { db } from '@/lib/db'
import { auth } from '@/lib/auth'

export async function createUser(data: { name: string; email: string }) {
  const session = await auth()
  if (!session?.user) throw new Error('Unauthorized')
  const user = await db.user.create({ data })
  return { id: user.id, name: user.name } // only return what UI needs
}

// Inline inside a Server Component
async function updatePost(formData: FormData) {
  'use server'
  await savePost(formData)
  revalidatePath('/posts')
}
```

### `'use cache'` (v16 — requires `cacheComponents: true`)
Caches a route, component, or function. Replaces `fetch()` cache options model.
```ts
// next.config.ts
const nextConfig: NextConfig = { cacheComponents: true }
```
```tsx
// File level
'use cache'
export default async function Page() { ... }

// Component level
export async function ProductList({ category }: { category: string }) {
  'use cache'
  const data = await db.products.findByCategory(category)
  return <ul>{data.map(p => <li key={p.id}>{p.name}</li>)}</ul>
}

// Function level
export async function getData() {
  'use cache'
  return fetch('/api/data').then(r => r.json())
}
```
**Cache key** = Build ID + Function ID + serializable arguments (including captured closure vars)

**Constraints:**
- Cannot call `cookies()`, `headers()`, `searchParams` inside — read outside and pass as args
- Args and return values must be serializable (strings, numbers, plain objects, arrays, Dates, Maps, Sets)
- Non-serializable values (children, Server Actions) can be passed through without introspecting them
- `React.cache` is isolated inside `use cache` boundaries

**Default revalidation:** stale 5min client, revalidate 15min server, never expires

```tsx
import { cacheLife, cacheTag, revalidateTag } from 'next/cache'

async function getProducts() {
  'use cache'
  cacheTag('products')      // tag for on-demand invalidation
  cacheLife('hours')        // built-in profile
  return fetch('/api/products').then(r => r.json())
}

// Server Action to invalidate:
export async function updateProduct() {
  'use server'
  await db.products.update(...)
  revalidateTag('products')
}
```

### `'use cache: private'` (experimental)
Like `use cache` but allows `cookies()`, `headers()`, `searchParams` inside. Results cached **only in browser memory**, never on server.
```tsx
async function getRecommendations(productId: string) {
  'use cache: private'
  cacheTag(`recs-${productId}`)
  cacheLife({ stale: 60 })
  const sessionId = (await cookies()).get('session-id')?.value || 'guest'
  return getPersonalizedRecs(productId, sessionId)
}
```

### `'use cache: remote'`
Stores entries in a remote handler (Redis/KV). Best for serverless where in-memory doesn't persist.
```tsx
async function getProductPrice(productId: string, currency: string) {
  'use cache: remote'
  cacheTag(`price-${productId}`)
  cacheLife({ expire: 3600 })
  return db.products.getPrice(productId, currency)
}
```
Use when: rate-limited APIs, slow backends, expensive computations, flaky services.

---

## Step 3: File-System Conventions (App Router)

### Core files

| File | Purpose |
|---|---|
| `layout.tsx` | Shared UI, persists across navigations, does NOT rerender on nav |
| `page.tsx` | Route UI, makes route publicly accessible |
| `loading.tsx` | Suspense fallback for the route segment |
| `error.tsx` | Error boundary — must be `'use client'` |
| `not-found.tsx` | Rendered when `notFound()` is called |
| `global-not-found.tsx` | Global 404 for unmatched URLs (experimental, must include `<html>` + `<body>`) |
| `forbidden.tsx` | Rendered when `forbidden()` is called — returns 403 (experimental) |
| `unauthorized.tsx` | Rendered when `unauthorized()` is called — returns 401 (experimental) |
| `global-error.tsx` | Error boundary for root layout (must include `<html>` + `<body>`) |
| `route.ts` | API Route Handler |
| `proxy.ts` | Runs before requests — replaces deprecated `middleware.ts` (v16) |
| `template.tsx` | Like layout but re-mounts on every navigation |
| `default.tsx` | Fallback for unmatched parallel route slots on hard navigation |
| `instrumentation.ts` | Server observability, `register()` runs once at server start |
| `instrumentation-client.ts` | Client observability, runs before React hydration |

### `params` and `searchParams` are Promises (v15+)

Always `await` them in Server Components:
```tsx
export default async function Page({
  params,
  searchParams,
}: {
  params: Promise<{ slug: string }>
  searchParams: Promise<{ [key: string]: string | string[] | undefined }>
}) {
  const { slug } = await params
  const { q = '' } = await searchParams
}
```
In Client Components, use React's `use()` hook:
```tsx
'use client'
import { use } from 'react'
export default function Page({ params }: { params: Promise<{ slug: string }> }) {
  const { slug } = use(params)
}
```
Use typed helpers (generated by `next dev`/`next build`):
- `PageProps<'/blog/[slug]'>` for pages
- `LayoutProps<'/dashboard'>` for layouts
- `RouteContext<'/api/[id]'>` for route handlers

### Dynamic routes
```
app/blog/[slug]/page.tsx           → /blog/a       → { slug: 'a' }
app/shop/[...slug]/page.tsx        → /shop/a/b     → { slug: ['a','b'] }
app/shop/[[...slug]]/page.tsx      → /shop         → { slug: undefined }
```

### Route Groups — `(folderName)`
Organizes routes without affecting URL. Use for multiple root layouts:
```
app/(marketing)/layout.tsx
app/(marketing)/page.tsx
app/(shop)/layout.tsx
app/(shop)/products/page.tsx
```
Navigating between different root layouts triggers a full page load.

### Parallel Routes — `@slotName`
Simultaneously render multiple pages in one layout:
```
app/
  layout.tsx              → receives { children, analytics, team }
  @analytics/
    page.tsx
  @team/
    page.tsx
    default.tsx           → required fallback for hard navigation
```
```tsx
export default function Layout({ children, analytics, team }: {
  children: React.ReactNode
  analytics: React.ReactNode
  team: React.ReactNode
}) {
  return <>{children}{analytics}{team}</>
}
```
Each slot can have independent `loading.tsx` and `error.tsx`. Use with conditional rendering for role-based UIs.

### Intercepting Routes — `(.)`, `(..)`, `(...)`, `(...)`
Load a route within current layout while masking the URL. Classic use: photo modals.
```
app/
  feed/page.tsx
  @modal/
    default.tsx           → returns null
    (.)photo/[id]/page.tsx  → intercepts /photo/[id] from feed
```
Combine with Parallel Routes for URL-shareable, refresh-safe modals.

### `error.tsx`
```tsx
'use client'
export default function Error({
  error,
  unstable_retry,   // v16.2+: use instead of reset()
}: {
  error: Error & { digest?: string }
  unstable_retry: () => void
}) {
  return (
    <div>
      <h2>Something went wrong!</h2>
      <button onClick={() => unstable_retry()}>Try again</button>
    </div>
  )
}
```
`error.tsx` does NOT wrap `layout.tsx` in the same segment. For root layout errors, use `global-error.tsx`.

### `template.tsx`
Like a layout but gets a unique key and **re-mounts on every navigation**. Use when `useEffect` needs to re-run or child state needs reset on route change.

### `instrumentation.ts`
```ts
export function register() {
  if (process.env.NEXT_RUNTIME === 'edge') require('./register.edge')
  else require('./register.node')
}
export async function onRequestError(err, request, context) {
  await fetch('https://.../report-error', {
    method: 'POST',
    body: JSON.stringify({ message: err.message, request, context }),
    headers: { 'Content-Type': 'application/json' },
  })
}
```

### `instrumentation-client.ts`
Runs after HTML loads, before React hydration. For analytics, error tracking, polyfills.
```ts
performance.mark('app-init')
window.addEventListener('error', (e) => reportError(e.error))

export function onRouterTransitionStart(url: string, navigationType: 'push' | 'replace' | 'traverse') {
  analytics.track('navigation', { url, type: navigationType })
}
```

---

## Step 4: Proxy (formerly Middleware)

> **⚠️ Breaking change in v16**: `middleware.ts` is **deprecated**. Rename file to `proxy.ts`, rename function from `middleware` to `proxy`.
> Migration codemod: `npx @next/codemod@canary middleware-to-proxy .`

```ts
// proxy.ts (at project root or inside src/)
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export function proxy(request: NextRequest) {
  const token = request.cookies.get('token')
  if (!token) {
    return NextResponse.redirect(new URL('/login', request.url))
  }
  return NextResponse.next()
}

export const config = {
  matcher: ['/dashboard/:path*'],
}
```

**Key APIs:**
- `NextResponse.redirect(url)` — redirect to new URL
- `NextResponse.rewrite(url)` — rewrite (user sees original URL)
- `NextResponse.next()` — continue to route
- `request.cookies.get/set/delete` — cookie access
- `request.nextUrl` — parsed URL with `pathname`, `searchParams`
- Modify request headers: `NextResponse.next({ request: { headers: newHeaders } })`

**Matcher options:**
```ts
export const config = {
  matcher: [
    // Exclude static files and API routes
    '/((?!api|_next/static|_next/image|favicon.ico).*)',
    // Complex: with has/missing conditions
    { source: '/api/:path*', has: [{ type: 'header', key: 'Authorization' }] },
  ],
}
```

**Execution order:** next.config headers → next.config redirects → Proxy → filesystem routes

**Important:** Always verify auth **inside** each Server Action too. Don't rely on Proxy alone.

---

## Step 5: Route Handlers (`app/api/route.ts`)

```ts
import { NextRequest, NextResponse } from 'next/server'
import { cookies, headers } from 'next/headers'

export async function GET(
  request: NextRequest,
  ctx: RouteContext<'/api/items/[id]'>
) {
  const { id } = await ctx.params
  return NextResponse.json({ id })
}

export async function POST(request: NextRequest) {
  const body = await request.json()
  return NextResponse.json({ received: body }, { status: 201 })
}
```

GET handlers default to **dynamic** in v15+. For ISR: `export const revalidate = 60`

---

## Step 6: Pages Router (legacy)

```tsx
export async function getStaticProps() {
  return { props: { data: await fetchData() }, revalidate: 60 }
}
export async function getServerSideProps() {
  return { props: { data: await fetchData() } }
}
export async function getStaticPaths() {
  return { paths: [{ params: { slug: 'hello' } }], fallback: 'blocking' }
}
```

---

## Step 7: Frontend Essentials

```tsx
// Always use next/image — never <img>
import Image from 'next/image'
<Image src="/hero.png" alt="Hero" width={1200} height={600} priority />

// Always use next/link for internal links
import Link from 'next/link'
<Link href="/about">About</Link>

// next/font — zero layout shift
import { Inter } from 'next/font/google'
const inter = Inter({ subsets: ['latin'] })

// Metadata
export const metadata: Metadata = { title: 'My App', description: '...' }
export async function generateMetadata({ params }): Promise<Metadata> {
  const { slug } = await params
  return { title: (await getPost(slug)).title }
}
```

**CSS:** CSS Modules > Tailwind > Global CSS (global only in layout/\_app)

---

## Step 8: Caching

### v16 model (Cache Components enabled)
Use `'use cache'` directives (see Step 2). `fetch()` cache options are ignored.

### v14/v15 legacy model
```tsx
await fetch('...', { next: { revalidate: 60 } })   // ISR
await fetch('...', { cache: 'no-store' })            // always dynamic
await fetch('...', { cache: 'force-cache' })         // always static

import { revalidatePath, revalidateTag } from 'next/cache'
revalidatePath('/blog')
revalidateTag('posts')
```

---

## Step 9: Scaffolding Protocol

1. **File tree first**
2. **Complete code for each file**
3. **Required installs**

```
my-app/
├── app/
│   ├── layout.tsx          # Root layout (<html> + <body> required)
│   ├── page.tsx
│   ├── globals.css
│   ├── error.tsx           # 'use client'
│   ├── not-found.tsx
│   └── [feature]/
│       ├── layout.tsx
│       ├── page.tsx
│       ├── loading.tsx
│       └── error.tsx
├── components/ui/
├── lib/
│   ├── db.ts
│   └── utils.ts
├── actions/                # 'use server' files
├── proxy.ts                # NOT middleware.ts in v16
├── instrumentation.ts
├── next.config.ts
└── tailwind.config.ts
```

---

## Step 10: Debugging Checklist

Check these first before deeper investigation:

- `params`/`searchParams` not awaited (Promises since v15)
- `middleware.ts` not renamed to `proxy.ts` (v16 breaking change)
- `'use client'` missing on component using hooks or events
- `'use server'` missing on a Server Action
- `cookies()`/`headers()` called inside `'use cache'` (must pass as args)
- `localStorage`/`window` used in a Server Component
- Non-serializable props passed Client→Server boundary
- `reset()` used instead of `unstable_retry()` in `error.tsx` (v16.2+)
- `dynamic`/`revalidate`/`fetchCache` route segment configs used with `cacheComponents: true` (removed in v16)
- Missing `default.tsx` for parallel route slots (causes 404 on hard navigation)

---

## Step 11: Reference Files

When a question requires deep detail, load the relevant reference file:

**`references/components.md`** — Full API for all built-in Next.js components:
- `next/image` — all props (`src`, `fill`, `sizes`, `preload`, `placeholder`, `blurDataURL`, `quality`, `loader`, `unoptimized`, `decoding`, `overrideSrc`, `onLoad`, `onError`), `getImageProps()`, full `next.config.js` image options (`remotePatterns`, `qualities`, `formats`, `deviceSizes`, `imageSizes`, `minimumCacheTTL`, `maximumDiskCacheSize`, etc.)
- `next/link` — all props (`href`, `replace`, `scroll`, `prefetch`, `onNavigate`, `transitionTypes`), active link detection, proxy rewrite pattern, navigation blocking pattern
- `next/font` — `next/font/google`, `next/font/local`, all options, CSS variable method, Tailwind v4/v3 integration, shared font definitions file, preloading scope rules
- `next/form` — string vs function `action`, `replace`/`scroll`/`prefetch` props, `useFormStatus` pending state, Server Action + redirect pattern, caveats
- `next/script` — all 4 load strategies (`beforeInteractive`, `afterInteractive`, `lazyOnload`, `worker`), `onLoad`/`onReady`/`onError` callbacks, rules per strategy

**`references/file-conventions.md`** — Full docs for every App Router file convention:
- `layout.tsx`, `page.tsx`, `loading.tsx`, `error.tsx`, `template.tsx`, `default.tsx`
- `not-found.tsx`, `global-not-found.tsx`, `forbidden.tsx`, `unauthorized.tsx`, `global-error.tsx`
- `route.ts`, `proxy.ts`, dynamic routes, route groups, parallel routes, intercepting routes
- Metadata files (OG images, sitemap, robots, manifest, icons)
- `instrumentation.ts` / `instrumentation-client.ts`, Route Segment Config, `public/`, `src/`

**`references/functions.md`** — Full API for every Next.js function and hook:
- **Server APIs**: `cookies()`, `headers()`, `draftMode()`, `connection()`, `after()`
- **Navigation**: `redirect()`, `permanentRedirect()`, `notFound()`, `forbidden()`, `unauthorized()`
- **Cache invalidation**: `revalidatePath()`, `revalidateTag()`, `updateTag()`, `refresh()`
- **Static generation**: `generateStaticParams()`, `generateMetadata()`, `generateViewport()`
- **Client hooks**: `useRouter()`, `usePathname()`, `useSearchParams()`, `useParams()`, `useSelectedLayoutSegment()`, `useLinkStatus()`, `useReportWebVitals()`
- **Error handling**: `unstable_rethrow()`, `unstable_catchError()`
- **Image generation**: `ImageResponse` (from `next/og`)
- **Proxy utilities**: `NextRequest`, `NextResponse`, `userAgent()`
- **Legacy/deprecated APIs** and what to use instead

**`references/cli.md`** — Full CLI reference for both tools:
- `create-next-app` — all flags, interactive prompts, linter options, bootstrapping from examples
- `next dev` — port, hostname, HTTPS, CPU profiling, Turbopack vs Webpack flags
- `next build` — debug options, `--debug-prerender`, `--debug-build-paths`, build output legend
- `next start` — `--keepAliveTimeout` for load balancer setups
- `next info`, `next telemetry`, `next typegen`, `next upgrade`, `next experimental-analyze`
- Node.js args, port configuration, CPU profiling patterns

**`references/turbopack.md`** — Turbopack bundler deep dive:
- Supported platforms and WASM fallback
- Full feature support matrix (language, CSS, assets, module resolution, magic comments)
- Known gaps vs Webpack: filesystem root, CSS module ordering, Sass tilde imports, build caching, webpack plugins
- Unsupported/unplanned features
- All `turbopack.*` and `experimental.turbopack*` config options with defaults
- Performance debugging with `NEXT_TURBOPACK_TRACING`

**`references/glossary.md`** — Definitions for all Next.js terminology:
- Load when the user asks "what is X?" for any Next.js-specific concept
- Covers: App Router, RSC Payload, hydration, prerendering, PPR, ISR, memoization, streaming, Suspense boundary, version skew, all directives, all routing patterns, and more

**`references/fast-refresh.md`** — Fast Refresh behavior and edge cases:
- How it decides between component update, multi-file update, or full reload
- Error resilience (syntax errors, runtime errors, error boundaries)
- When state resets: class components, HOCs, anonymous arrow functions, mixed exports
- Hook behavior during refresh (`useState`/`useRef` preserved; `useEffect`/`useMemo`/`useCallback` always re-run)
- `// @refresh reset` directive, case-sensitive import gotcha

**`references/nextjs-compiler.md`** — SWC-based compiler configuration:
- Load when asked about Babel replacement, SWC, minification, or `compiler.*` config options
- Styled Components, Jest, Relay, Emotion setup
- `removeConsole`, `reactRemoveProperties`, `define`/`defineServer`, `transpilePackages`
- Legacy decorators, `jsxImportSource`, `runAfterProductionCompile` hook
- SWC trace profiling, SWC plugins (WASM), Babel fallback behavior

**`references/rspack.md`** — Community Rspack plugin for Next.js:
- Experimental, not production-ready
- Links to docs, example, and feedback thread

**`references/edge-runtime.md`** — Edge Runtime vs Node.js Runtime reference:
- Load when asked about Edge Runtime support, `proxy.ts` runtime, or whether a specific API/package works in Edge
- Side-by-side runtime comparison table (ISR, filesystem, `require`, `node_modules` constraints)
- Full list of supported APIs by category (Network, Encoding, Streams, Crypto, Web Standards)
- Unsupported APIs: all native Node.js APIs, `require()`, `eval`, `new Function`, `WebAssembly.compile/instantiate`
- `unstable_allowDynamic` escape hatch for packages with unreachable dynamic eval
- Common gotchas: no `Buffer`, no `process.nextTick`, no `crypto` (Node) → use Web Crypto, package compatibility tips
- How to opt a Route Handler into Edge Runtime (`export const runtime = 'edge'`)

**`references/directives.md`** — Full reference for all five Next.js directives:
- Load when asked for deep detail on any directive behavior, serialization rules, nesting rules, or edge cases
- `'use client'` — entry point rules, serialization constraints, composition patterns
- `'use server'` — file-level vs inline, using Server Actions in Client Components, security rules (auth, return value constraints)
- `'use cache'` — cache key mechanics (Build ID + Function ID + args + closures), serialization supported/unsupported types, pass-through pattern for children/Server Actions, `cacheLife`/`cacheTag`/`revalidateTag` usage, full revalidation model, route-level caching, build hang troubleshooting
- `'use cache: private'` (experimental) — browser-only caching, allowed runtime APIs (`cookies`, `headers`, `searchParams`), nesting restrictions
- `'use cache: remote'` — remote handler setup, cache key cardinality strategy, three-directive comparison table, nesting rules, `connection()` deferred execution pattern

**`references/configuration.md`** — Full `next.config.ts` options reference + TypeScript + ESLint:
- Load when asked about any specific `next.config.*` option, `next.config.ts` file format, phase-aware config, or unit testing config
- Complete A–Z table of all config options with descriptions
- Deep detail on key options: `typescript`, `typedRoutes`, `images`, `headers`/`redirects`/`rewrites`, `serverActions`, `output`, `cacheComponents`
- TypeScript deep dive: `next-env.d.ts`, `typedEnv`, route-aware type helpers (`PageProps`, `LayoutProps`, `RouteContext`), end-to-end type safety, `next typegen`
- ESLint setup (v16+): `next lint` removed, flat config, `eslint-config-next` variants, Prettier integration, monorepo config, lint-staged

**`references/fetching-data.md`** — Data fetching patterns (Server + Client Components):
- Load when asked about fetching data, `fetch` API, ORM/database queries in Server Components, streaming, `loading.js`, `<Suspense>`, `use()` hook, SWR, React Query
- Sequential vs parallel data fetching, `Promise.all` / `Promise.allSettled` patterns
- Sharing data across the tree with `React.cache` + context providers
- When `loading.js` doesn't cover layout-level uncached access and how to fix it

**`references/css.md`** — Styling options and CSS ordering:
- Load when asked about Tailwind CSS setup, CSS Modules, global CSS, external stylesheets, CSS ordering/merging, dev vs production CSS behavior
- Tailwind v4 install (`@tailwindcss/postcss`), PostCSS config, global import pattern
- CSS Modules scoping, naming conventions
- Recommendations for when to use Tailwind vs CSS Modules vs global CSS
- `cssChunking` config option

**`references/deploying.md`** — Deployment options and platform adapters:
- Load when asked about deploying, Node.js server, Docker, static export, Vercel, Cloudflare, Netlify, adapters, self-hosting
- Feature support table per deployment option
- Docker templates (`standalone`, `export`, multi-env)
- Verified adapters (Vercel, Bun) vs other platform integrations
- Links to provider-specific guides (DigitalOcean, Fly.io, GCR, Render, SST, Amplify, Firebase, Deno, Appwrite)

**`references/installation.md`** — Installation, project setup, and configuration:
- Load when asked about getting started, `create-next-app`, manual installation, TypeScript setup, linting (ESLint/Biome), path aliases, `src/` directory, `public/` folder, system requirements, supported browsers
- Quick start commands for pnpm/npm/yarn/bun
- Interactive CLI prompts and all options (React Compiler, `AGENTS.md`, import alias)
- Manual setup: `app/layout.tsx`, `app/page.tsx`, `package.json` scripts
- TypeScript VS Code plugin activation, `tsconfig.json` `paths` / `baseUrl` setup
- Note: `next build` no longer auto-runs linter in v16

**`references/upgrading.md`** — Upgrading Next.js versions:
- Load when asked about upgrading Next.js, the `upgrade` command, canary releases, version migration, codemods
- `next upgrade` command (v16.1.0+), fallback for older versions
- Manual upgrade steps including `eslint-config-next`
- Canary features currently gated (forbidden/unauthorized auth APIs)
- Links to version guides: v16, v15, v14

**`references/accessibility.md`** — Built-in accessibility features:
- Load when asked about accessibility, a11y, screen readers, route announcements, ARIA, `eslint-plugin-jsx-a11y`, WCAG, `prefers-reduced-motion`
- Route announcer: how it determines page name (title → h1 → pathname)
- ESLint a11y rules included by default (aria-props, role validation, alt text)
- External resources: WebAIM, WCAG 2.2, A11y Project, color contrast, reduced motion

---

## Step 11b: Guides Reference Files

All guide files live in `references/guides/`. Load the relevant file when the question matches.

**`references/guides/ai-coding-agents.md`** — `AGENTS.md` / `CLAUDE.md` setup for AI coding agents:
- Load when asked about AGENTS.md, CLAUDE.md, AI agents, coding assistants, bundled docs, `node_modules/next/dist/docs/`

**`references/guides/analytics.md`** — Web Vitals and analytics setup:
- Load when asked about `useReportWebVitals`, `instrumentation-client.js`, Web Vitals (FCP, LCP, CLS, FID, INP, TTFB), Google Analytics integration, sending metrics to external systems

**`references/guides/authentication.md`** — Full auth implementation guide:
- Load when asked about authentication, login, signup, sessions, cookies, JWT, stateless sessions, database sessions, DAL (Data Access Layer), DTOs, auth with Proxy, `verifySession`, auth libraries (Clerk, NextAuth, Auth0, etc.)

**`references/guides/backend-for-frontend.md`** — Route Handlers as API layer (BFF pattern):
- Load when asked about Route Handlers, public API endpoints, content types, content negotiation, webhooks, callback URLs, `NextRequest`/`NextResponse`, rate limiting, proxying requests, `proxy.ts` as API gateway

**`references/guides/caching-previous-model.md`** — Caching without Cache Components (`cacheComponents` flag off):
- Load when asked about `fetch` caching, `unstable_cache`, `dynamic` route config, `fetchCache`, `revalidate` segment config, `revalidateTag`, `revalidatePath`, `React.cache` deduplication, preloading data — for projects NOT using `cacheComponents: true`

**`references/guides/cdn-caching.md`** — CDN integration and cache headers:
- Load when asked about CDN caching, `Cache-Control` headers, `s-maxage`, `stale-while-revalidate`, `Vary` header, `_rsc` query param, `next-router-state-tree`, on-demand revalidation with CDN, `assetPrefix`, pathname-based cache keying direction

**`references/guides/ci-build-caching.md`** — CI pipeline cache configuration:
- Load when asked about CI build caching, `.next/cache`, GitHub Actions, CircleCI, GitLab CI, Vercel, Travis CI, Netlify CI, AWS CodeBuild, Bitbucket, Heroku, Azure Pipelines, Jenkins

**`references/guides/content-security-policy.md`** — CSP with nonces and SRI:
- Load when asked about Content Security Policy, CSP, nonces, `unsafe-inline`, `unsafe-eval`, SRI (Subresource Integrity), XSS protection, `script-src`, dynamic rendering requirement for nonces, PPR + CSP incompatibility

**`references/guides/css-in-js.md`** — CSS-in-JS libraries in App Router:
- Load when asked about styled-components, styled-jsx, emotion, Chakra UI, MUI, Tailwind (CSS-in-JS variant), `useServerInsertedHTML`, style registry pattern for SSR

**`references/guides/custom-server.md`** — Programmatic Next.js server:
- Load when asked about custom server, `next()` function, `server.js`, programmatic server setup, `app.prepare()`, `handle(req, res)`

**`references/guides/data-security.md`** — Data security patterns and best practices:
- Load when asked about data security, preventing data exposure to client, Zero Trust, DAL, DTOs, taint APIs, `server-only` package, Server Action security, CSRF, encryption keys, IDOR vulnerabilities, auditing Next.js apps

**`references/guides/debugging.md`** — Debugging with VS Code, Chrome, Firefox:
- Load when asked about debugging, VS Code launch.json, `--inspect` flag, Chrome DevTools for server-side code, React DevTools, JetBrains WebStorm, Windows Defender slow dev

**`references/guides/deploying-to-platforms.md`** — Platform support matrix and adapter API:
- Load when asked about platform support, functional fidelity vs performance fidelity, feature matrix (streaming, shared cache, edge stitching), CDN infrastructure table, verified adapters, `cacheHandler` vs `cacheHandlers`, adapter API

**`references/guides/draft-mode.md`** — Previewing CMS draft content:
- Load when asked about Draft Mode, `draftMode()`, headless CMS preview, `__prerender_bypass` cookie, enabling draft mode via Route Handler

**`references/guides/environment-variables.md`** — Environment variable loading and exposure:
- Load when asked about `.env` files, `NEXT_PUBLIC_` prefix, runtime env vars, `@next/env`, `loadEnvConfig`, variable expansion, test environment, `NODE_ENV`, env load order

**`references/guides/forms.md`** — Forms with Server Actions:
- Load when asked about forms, Server Actions for form submissions, `useActionState`, `useFormStatus`, form validation with Zod, optimistic updates, `useOptimistic`, programmatic form submission, `requestSubmit()`

**`references/guides/how-revalidation-works.md`** — Internal revalidation architecture for platform engineers:
- Load when asked about how revalidation works internally, tag system architecture, soft tags, `_N_T_` prefix, multi-instance cache coordination, `updateTags()`, `refreshTags()`, custom cache handler implementation, HTML/RSC consistency

**`references/guides/incremental-static-regeneration.md`** — ISR setup and patterns:
- Load when asked about ISR, Incremental Static Regeneration, `revalidate` export, `generateStaticParams`, `revalidateTag`, `revalidatePath`, `unstable_cache` with tags, on-demand revalidation, `x-nextjs-cache` header, ISR caveats

**`references/guides/instrumentation.md`** — Server startup instrumentation:
- Load when asked about instrumentation, `instrumentation.ts`, `register()` function, server startup code, side effects on startup, runtime-specific imports (`NEXT_RUNTIME`)

**`references/guides/internationalization.md`** — i18n routing and localization:
- Load when asked about internationalization, i18n, locales, `Accept-Language`, `proxy.js` locale detection, `app/[lang]`, dictionaries, `getDictionary`, `generateStaticParams` for locales, `next-intl`, `lingui`

**`references/guides/json-ld.md`** — Structured data with JSON-LD:
- Load when asked about JSON-LD, structured data, schema.org, rich results, `schema-dts`, `dangerouslySetInnerHTML` for JSON-LD, XSS prevention with `\u003c`

**`references/guides/lazy-loading.md`** — Code splitting and lazy loading:
- Load when asked about lazy loading, `next/dynamic`, `React.lazy`, `ssr: false`, dynamic imports, named export dynamic imports, `webpackIgnore`, `turbopackIgnore`, `turbopackOptional`, loading component for dynamic imports

**`references/guides/local-development.md`** — Dev environment performance:
- Load when asked about slow development server, Fast Refresh performance, antivirus exclusions, Turbopack dev, barrel files, `optimizePackageImports`, Tailwind content scanning, Turbopack tracing, `NEXT_TURBOPACK_TRACING`, Docker on Mac performance

**`references/guides/mcp-server.md`** — Next.js MCP Server for coding agents:
- Load when asked about Next.js MCP, `next-devtools-mcp`, `.mcp.json`, `get_errors`, `get_routes`, `get_page_metadata`, real-time app state for AI agents

**`references/guides/mdx.md`** — MDX setup and usage:
- Load when asked about MDX, `@next/mdx`, `mdx-components.tsx`, frontmatter, remark plugins, rehype plugins, dynamic MDX imports, MDX with Tailwind typography, Rust-based MDX compiler

**`references/guides/memory-usage.md`** — Build and runtime memory optimization:
- Load when asked about memory usage, out of memory errors, heap snapshots, `--experimental-debug-memory-usage`, `webpackMemoryOptimizations`, webpack build worker, disable source maps, `preloadEntriesOnStart`, Edge memory issues

**`references/guides/migrating-to-cache-components.md`** — Migration from route segment configs to `use cache`:
- Load when asked about migrating to Cache Components, replacing `dynamic = 'force-static'`, replacing `revalidate` export, replacing `fetchCache`, `runtime = 'edge'` with `cacheComponents`

**`references/guides/migrating/index.md`** — Migration options overview:
- Load for general migration questions; see sub-files for specifics

**`references/guides/migrating/app-router.md`** — Pages Router → App Router migration:
- Load when asked about migrating from Pages Router to App Router, converting `getServerSideProps`/`getStaticProps`, `_app.js`/`_document.js` migration, `useRouter` API differences

**`references/guides/migrating/from-create-react-app.md`** — CRA → Next.js migration:
- Load when asked about migrating from Create React App, CRA to Next.js, `output: 'export'` SPA mode, catch-all route entry point, `ssr: false` for client-only components

**`references/guides/migrating/from-vite.md`** — Vite → Next.js migration:
- Load when asked about migrating from Vite, `import.meta.env`, `VITE_` prefix to `NEXT_PUBLIC_`, Vite config to Next.js config, SPA migration from Vite

**`references/guides/multi-tenant.md`** — Multi-tenant architecture:
- Load when asked about multi-tenant apps, subdomain routing, per-tenant data isolation, Platforms Starter Kit

**`references/guides/multi-zones.md`** — Micro-frontends with Multi-Zones:
- Load when asked about multi-zones, micro-frontends, `assetPrefix`, cross-zone navigation, zone routing with rewrites, monorepo zones

**`references/guides/open-telemetry.md`** — OpenTelemetry setup and custom spans:
- Load when asked about OpenTelemetry, `@vercel/otel`, `registerOTel`, `NodeSDK`, custom spans, `trace.getTracer`, built-in Next.js spans, `NEXT_OTEL_VERBOSE`, `NEXT_OTEL_FETCH_DISABLED`

**`references/guides/package-bundling.md`** — Bundle analysis and optimization:
- Load when asked about bundle analysis, `experimental-analyze`, `@next/bundle-analyzer`, `optimizePackageImports`, `serverExternalPackages`, moving computation to Server Components to reduce client bundle

**`references/guides/ppr-platform-guide.md`** — PPR implementation for platform engineers:
- Load when asked about implementing PPR on a platform, `postponedState`, resume protocol, `next-resume` header, CDN shell + origin compute architecture, `onCacheEntryV2`, adapter PPR implementation

**`references/guides/prefetching.md`** — Prefetching configuration and patterns:
- Load when asked about prefetching, `prefetch` prop on `<Link>`, `router.prefetch()`, hover prefetch, `onInvalidate`, `staleTimes` config, prefetch scheduling, preventing too many prefetches

**`references/guides/preserving-ui-state.md`** — Preserving React and DOM state across navigations:
- Load when asked about preserving UI state, Activity component, `display: none` navigation, form state across routes, dropdown reset on navigation, `useLayoutEffect` cleanup for Activity, `cacheComponents: true` required

**`references/guides/production.md`** — Pre-production checklist:
- Load when asked about production checklist, performance best practices, security checklist, metadata/SEO checklist, Core Web Vitals, Lighthouse, bundle analysis before deploy

**`references/guides/progressive-web-apps.md`** — PWA setup with Next.js:
- Load when asked about PWA, Progressive Web App, web app manifest, `manifest.ts`, Web Push notifications, VAPID keys, service worker (`sw.js`), install prompt, iOS home screen

**`references/guides/public-pages.md`** — Building static/public pages with Cache Components:
- Load when asked about public pages, static pages with shared data, Cache Components pattern, `'use cache'` for product lists/blogs, partial prerendering for mostly-static pages

**`references/guides/redirecting.md`** — All redirect methods:
- Load when asked about redirects, `redirect()`, `permanentRedirect()`, `redirects` in config, `NextResponse.redirect`, large-scale redirects, Bloom filter redirects, 307/308 status codes

**`references/guides/rendering-philosophy.md`** — Next.js rendering model explained:
- Load when asked about Next.js rendering philosophy, static vs dynamic spectrum, component-level boundaries, why PPR requires streaming, functional fidelity vs performance fidelity

**`references/guides/sass.md`** — Sass/SCSS setup:
- Load when asked about Sass, SCSS, `.module.scss`, `sassOptions`, `additionalData`, `sass-embedded`, exporting Sass variables to JS

**`references/guides/scripts.md`** — Optimizing third-party scripts:
- Load when asked about `next/script`, script loading strategies (`beforeInteractive`, `afterInteractive`, `lazyOnload`, `worker`), Partytown, inline scripts, `onLoad`/`onReady`/`onError` handlers

**`references/guides/self-hosting.md`** — Self-hosting configuration guide:
- Load when asked about self-hosting, nginx reverse proxy, custom cache handler, `cacheHandler`, build cache (`generateBuildId`), multi-server encryption key, `deploymentId`, graceful shutdown, `after()`, streaming with nginx, CDN with self-hosted Next.js

**`references/guides/single-page-applications.md`** — SPA patterns in Next.js:
- Load when asked about SPAs, single-page applications, `use()` API with context, SWR with Server Components, `output: 'export'` static SPA, shallow routing, `window.history.pushState`, client-only components, migrating from CRA/Vite

**`references/guides/static-exports.md`** — Static HTML export configuration:
- Load when asked about static export, `output: 'export'`, `next export` (removed), supported/unsupported features in static export, deploying to S3/Nginx/GitHub Pages, custom image loader for static export

**`references/guides/streaming.md`** — Streaming deep dive:
- Load when asked about streaming internals, HTTP contract for streaming, static shell, component payload, RSC payload, Web Vitals impact (TTFB/FCP/LCP/CLS/INP), Route Handler streaming with `ReadableStream`, verifying streaming works, nginx buffering

**`references/guides/tailwind-v3.md`** — Tailwind CSS v3 setup (broader browser support):
- Load when asked about Tailwind v3, `tailwindcss@^3`, PostCSS config for v3, `@tailwind base/components/utilities` directives. For v4, use `references/css.md` instead.

**`references/guides/testing/index.md`** — Testing overview:
- Load for general testing questions; routes to specific testing tool files

**`references/guides/testing/cypress.md`** — Cypress E2E and component testing:
- Load when asked about Cypress, E2E testing, Cypress component tests, `cy.visit`, `cy.mount`, `cypress.config.ts`, headless CI with Cypress

**`references/guides/testing/jest.md`** — Jest unit and snapshot testing:
- Load when asked about Jest, `next/jest`, `jest.config.ts`, `@testing-library/react`, `jest-environment-jsdom`, `@testing-library/jest-dom`, snapshot tests, module path aliases in Jest

**`references/guides/testing/playwright.md`** — Playwright E2E testing:
- Load when asked about Playwright, `@playwright/test`, `playwright.config.ts`, `page.goto`, `page.click`, `expect(page).toHaveURL`, visibility-aware selectors with `getByRole`, Activity boundary testing

**`references/guides/testing/vitest.md`** — Vitest unit testing:
- Load when asked about Vitest, `vitest.config.mts`, `@vitejs/plugin-react`, `vite-tsconfig-paths`, `jsdom` test environment, `@testing-library/react` with Vitest

**`references/guides/third-party-libraries.md`** — `@next/third-parties` optimized integrations:
- Load when asked about `@next/third-parties`, Google Tag Manager, Google Analytics GA4, `sendGTMEvent`, `sendGAEvent`, Google Maps Embed, YouTube Embed, `lite-youtube-embed`

**`references/guides/videos.md`** — Video embedding and self-hosting:
- Load when asked about video, `<video>` tag, `<iframe>` embedding, Vercel Blob for videos, `autoPlay`/`muted`/`playsInline` attributes, video subtitles, `react-player`, Mux, Cloudinary video, `next-video`

**`references/guides/upgrading/index.md`** — Upgrading overview:
- Load for general upgrading questions; routes to specific version files

**`references/guides/upgrading/codemods.md`** — All available codemods:
- Load when asked about codemods, `@next/codemod`, `upgrade` command, `middleware-to-proxy`, `next-async-request-api`, `next-og-import`, `built-in-next-font`, `new-link`, specific codemod names

**`references/guides/upgrading/version-14.md`** — v13 → v14 upgrade:
- Load when asked about upgrading to v14, Node.js 18.17 requirement, `next export` removal, `next/og` ImageResponse

**`references/guides/upgrading/version-15.md`** — v14 → v15 upgrade:
- Load when asked about upgrading to v15, async `cookies()`/`headers()`/`draftMode()`, async `params`/`searchParams`, `UnsafeUnwrapped` types, React 19 migration, `useActionState`, `staleTimes`, fetch no longer cached by default

**`references/guides/upgrading/version-16.md`** — v15 → v16 upgrade:
- Load when asked about upgrading to v16, `middleware.ts` → `proxy.ts`, `next lint` removed, Turbopack default, `experimental_ppr` removed, `unstable_` prefix removal, Cache Components, MCP server, `AGENTS.md`

---

## Step 12: When to Fetch Live Docs

Baked-in knowledge first. Fetch `https://nextjs.org/docs` only when:
- User is on a specific version and behavior might differ
- Question involves a niche or very new API
- User explicitly asks to verify against docs

Use `web_fetch` on the specific page, not a broad search.

---

## Step 13: CLI Reference

```bash
# Dev server (HMR + Fast Refresh)
next dev                        # Webpack (legacy default)
next dev --turbopack            # Turbopack (recommended, stable in v15+)
next dev --port 4000            # Custom port
next dev --hostname 0.0.0.0    # Expose to network

# Production build
next build                      # Full build
next build --turbopack          # Turbopack build (experimental in v15, maturing in v16)

# Start production server
next start
next start --port 4000

# Lint
next lint                       # Runs ESLint with Next.js rules
next lint --fix                 # Auto-fix

# Info (for bug reports / env diagnostics)
next info                       # Prints OS, Node, Next.js, browser info

# Codebases
npx @next/codemod@canary middleware-to-proxy .   # v16: rename middleware → proxy
npx @next/codemod@canary upgrade latest          # Auto-upgrade to latest
```

**Environment variables in CLI context:**
- `NODE_ENV=production next build` — force production mode
- `ANALYZE=true next build` — bundle analyzer (requires `@next/bundle-analyzer`)
- `NEXT_TELEMETRY_DISABLED=1` — opt out of telemetry

---

## Step 14: Bundlers — Turbopack, Webpack, Rspack

### Turbopack (default in v16 dev)
Rust-based incremental bundler built into Next.js. The `next dev --turbopack` flag is **stable as of Next.js 15**; build support (`next build --turbopack`) is experimental and maturing in v16.

```ts
// next.config.ts — enable Turbopack build (experimental)
const nextConfig: NextConfig = {
  experimental: {
    turbo: {
      rules: {
        '*.svg': { loaders: ['@svgr/webpack'], as: '*.js' },
      },
      resolveAlias: {
        // Map module names to different paths
        'lodash': 'lodash-es',
      },
    },
  },
}
```

**Turbopack vs Webpack tradeoffs:**
| | Turbopack | Webpack |
|---|---|---|
| Dev startup | Significantly faster (compiles only what's requested) | Slower (full bundle upfront) |
| HMR | Faster (fine-grained graph updates) | Slower |
| Build | Experimental | Stable, battle-tested |
| Plugin ecosystem | Limited (no Webpack plugins) | Vast |
| Custom loaders | Via `turbo.rules` (subset) | Full Webpack loader API |

**When to stick with Webpack:** custom Webpack plugins, complex loader chains, or when Turbopack build parity is needed for production.

### Rspack
Rust-based Webpack-compatible bundler from ByteDance. **Not built into Next.js** — used via community adapter [`@rspack/core`](https://rspack.dev/) or the `next-rspack` experimental integration. Offers ~5–10× faster builds than Webpack while maintaining near-full Webpack plugin/loader compatibility.

```ts
// next.config.ts — experimental Rspack integration (community, not official)
// Requires: npm install @rspack/core next-rspack
import { withRspack } from 'next-rspack'
export default withRspack({})
```

**Rspack vs Turbopack:**
- **Rspack**: Drop-in Webpack replacement, compatible with most Webpack plugins. Good for migrating large Webpack configs.
- **Turbopack**: Native Next.js integration, tighter optimizations, but requires rewriting Webpack-specific config.

**Recommendation:** Use Turbopack for greenfield Next.js projects. Use Rspack if you have a heavy Webpack plugin dependency that doesn't have a Turbopack equivalent yet.

---

## Step 15: Next.js Compiler (SWC)

Next.js uses **SWC** (Speedy Web Compiler, written in Rust) as its default compiler since v12. It replaces Babel and is ~17× faster.

### What SWC handles
- TypeScript stripping
- JSX transform
- Minification (replaces Terser)
- Module transforms (CommonJS ↔ ESM)
- Dead code elimination
- Built-in transforms (styled-components, emotion, relay, etc.)

### Configuring in `next.config.ts

- TypeScript by default — proper types, avoid `any`
- `proxy.ts` not `middleware.ts` in v16 projects
- `await params` / `await searchParams` — Promises since v15
- Server-first — `'use client'` only when needed, push boundary down
- No `useEffect` for data fetching — fetch in Server Components
- Handle loading + error states in all scaffolded features
- `next/image` over `<img>`, `next/link` over `<a>` for internal links
- `NEXT_PUBLIC_` prefix for client env vars, never expose secrets
- Auth inside Server Actions — don't rely on Proxy alone