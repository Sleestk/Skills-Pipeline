# Next.js Glossary (v16.2.1)

---

## A

**App Router** — The Next.js router introduced in v13, built on React Server Components. File-system based routing with support for layouts, nested routing, loading states, and error handling.

---

## B

**Build time** — When Next.js compiles code into optimized production files, generates static pages, and prepares assets. See `next build`.

---

## C

**Cache Components** — Feature enabling component/function-level caching via the `"use cache"` directive. Allows mixing static, cached, and dynamic content in one route. Prerendered HTML shell served immediately; dynamic content streams in. Configure with `cacheLife()`, tag with `cacheTag()`, invalidate with `updateTag()`.

**Catch-all Segments** — Dynamic route segments matching multiple URL parts via `[...folder]/page.js` syntax. Useful for docs sites, file browsers.

**Client Bundles** — JavaScript bundles sent to the browser. Next.js splits these automatically by route based on the module graph.

**Client Component** — React component that runs in the browser. Can also be server-rendered during initial generation. Supports state, effects, event handlers, browser APIs. Marked with `"use client"` directive.

**Client Cache** — In-memory browser cache storing RSC Payload for visited/prefetched routes. Serves cached layouts and loading states instantly without server requests. Pages not cached by default but reused on back/forward nav. Cleared on page refresh.

Invalidated by: `revalidateTag`, `revalidatePath`, `updateTag`, `router.refresh`, `cookies.set`, `cookies.delete`.

Configure duration: `staleTimes` (global) or `stale` property in `cacheLife` (per-route, recommended).

**Client-side navigation** — Page content updates without a full reload. Used by `<Link>` component, preserving shared layout state and browser state.

**Code Splitting** — Dividing the app into smaller JS chunks by route. Only the code for the current route loads, reducing initial load time. Done automatically by Next.js.

---

## D

**Dynamic rendering** — Component rendered at request time (not build time). Triggered by use of Request-time APIs.

**Dynamic route segments** — Route segments generated from data at request time. Created by wrapping a folder name in brackets: `[slug]`.

---

## E

**Environment Variables** — `NEXT_PUBLIC_` prefix exposes variables to the browser; all others are server-only.

**Error Boundary** — React component catching JS errors in its subtree, displaying fallback UI. Created via `error.js` file (must be `'use client'`).

---

## F

**Fast Refresh** — Hot module reloading that gives instantaneous feedback on edits while preserving React component state. Default in all Next.js apps v9.4+. See `references/fast-refresh.md` for full detail.

**File-system caching** — Turbopack feature storing compiler artifacts on disk between runs. Reduces recompile times across `next dev` and `next build` sessions.

**Font Optimization** — Automatic via `next/font`. Self-hosts fonts, eliminates layout shift, handles Google Fonts and local files.

---

## H

**Hydration** — React's process of attaching event handlers to server-rendered HTML to make it interactive. React reconciles server markup with client JS.

---

## I

**Import Aliases** — Custom path mappings for frequently used directories (e.g. `@/components`). Reduces relative import complexity.

**Incremental Static Regeneration (ISR)** — Updates static content without rebuilding the entire site. Revalidates pages in the background as traffic comes in. Also called Revalidation in Next.js.

**Intercepting Routes** — Loads a route from another part of the app within the current layout (e.g. modals). Keeps URL shareable without switching context.

**Image Optimization** — Automatic via `<Image>` component. Optimizes on-demand, serves WebP, handles lazy loading and responsive sizing.

---

## L

**Layout** — Shared UI across multiple pages. Persists state, stays interactive, does not re-render on navigation. Defined in `layout.js`.

**Loading UI** — Fallback shown while a route segment loads. Created via `loading.js` — automatically wraps the page in a Suspense boundary.

---

## M

**Module Graph** — Graph of file dependencies. Each file is a node; import/export relationships are edges. Next.js uses this to determine bundling and code-splitting.

**Metadata** — Page info for browsers and search engines (title, description, OG images). Defined via `metadata` export or `generateMetadata` function.

**Memoization** — Caches a function's return value so repeated calls during a render pass execute it only once. `fetch` GET requests with the same URL/options are automatically memoized across Server Components, layouts, pages, and `generateMetadata`/`generateStaticParams`. Use React `cache()` for non-fetch operations.

**Middleware** — See Proxy.

---

## N

**Not Found** — Component shown when a route doesn't exist or `notFound()` is called. Created via `not-found.js`.

---

## P

**Private Folders** — Folders prefixed with `_` (e.g. `_components`). Excluded from the routing system; used for code organization.

**Page** — UI unique to a route. Exported from `page.js` within `app/`.

**Parallel Routes** — Simultaneously or conditionally render multiple pages in one layout. Uses `@folder` named slots convention.

**Partial Prerendering (PPR)** — Combines prerendering and dynamic rendering in one route. Static shell served immediately; dynamic content streams in. Best of both strategies.

**Prefetching** — Loading a route in the background before navigation. `<Link>` automatically prefetches routes when they enter the viewport.

**Prerendering** — Component rendered at build time or during background revalidation. Produces HTML + RSC Payload, cached and served from CDN. Default for components not using Request-time APIs. Also called Static rendering.

**Proxy** — File (`proxy.js`) running server-side code before a request completes. Used for logging, redirects, rewrites. Formerly called Middleware. Replaces `middleware.ts` in v16.

---

## R

**Redirect** — Sends users from one URL to another. Configurable in `next.config.js`, returned from Proxy, or triggered with `redirect()`.

**Request-time APIs** — Functions accessing request-specific data; opts a component into dynamic rendering:
- `cookies()` — request cookies
- `headers()` — request headers
- `searchParams` — URL query params
- `draftMode()` — draft mode

**Runtime rendering** — See Dynamic rendering.

**Revalidation** — Updating cached data. Time-based via `cacheLife()` or on-demand via `cacheTag()` + `updateTag()`. Also known as ISR.

**Rewrite** — Maps an incoming path to a different destination without changing the browser URL. Configured in `next.config.js` or returned from Proxy.

**Route Groups** — Organizes routes without affecting URL structure. Folder name wrapped in parentheses: `(marketing)`. Enables per-group layouts.

**Route Handler** — Function handling HTTP requests for a specific route, defined in `route.js`. Uses Web Request/Response APIs. Supports GET, POST, PUT, PATCH, DELETE, HEAD, OPTIONS.

**Route Segment** — A part of the URL path (between two slashes) defined by a folder in `app/`.

**RSC Payload** — Compact binary representation of the rendered React Server Components tree. Contains Server Component render results, Client Component placeholders, and cross-boundary props.

---

## S

**Server Component** — Default component type in App Router. Renders on the server, fetches data directly, adds nothing to the client JS bundle. Cannot use state or browser APIs.

**Server Action** — A Server Function passed to a Client Component as a prop or bound to a form action. Commonly used for form submissions and data mutations.

**Server Function** — Async function running on the server, marked with `"use server"` directive. Can be invoked from Client Components.

**Static Export** — Generates a fully static site (HTML/CSS/JS). Enabled via `output: 'export'` in `next.config.js`. Hostable on any static file server; no Node.js required.

**Static rendering** — See Prerendering.

**Static Assets** — Images, fonts, videos served directly. Stored in `public/`, referenced by relative paths.

**Static Shell** — Prerendered HTML structure of a page served immediately. With PPR, includes all static content plus Suspense fallbacks for dynamic content.

**Streaming** — Server sends page parts to the client as they become ready. Enabled automatically by `loading.js` or manual `<Suspense>` boundaries.

**Suspense boundary** — React `<Suspense>` component wrapping async content with fallback UI. Defines where the static shell ends and streaming begins; enables PPR.

---

## T

**Turbopack** — Fast, Rust-based bundler built into Next.js. Default for `next dev` and `next build`. Significantly faster than Webpack. See `references/turbopack.md`.

**Tree Shaking** — Removes unused code from JS bundles during build. Done automatically by Next.js.

---

## U

**`"use cache"` directive** — Marks a component or function as cacheable. File-level or inline at function scope.

**`"use client"` directive** — Marks the server/client boundary. Must be at top of file, before imports. All exports + imported dependencies go into the client bundle.

**`"use server"` directive** — Marks a function as a Server Function callable from client-side code. File-level or inline at function scope.

---

## V

**Version skew** — Mismatch between client and server versions after a new deployment. Clients may reference JS/CSS/data from an older build, causing missing assets, Server Action errors, navigation failures. Next.js uses `deploymentId` to detect and handle this.