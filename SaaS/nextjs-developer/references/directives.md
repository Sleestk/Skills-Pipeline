# Next.js Directives — Full Reference

> Version: 16.2.1 | Source: https://nextjs.org/docs/app/api-reference/directives

All directives are React features surfaced through Next.js. Available directives:
- `'use client'` — render component on the client
- `'use server'` — execute function/file on the server (Server Actions)
- `'use cache'` — cache a route, component, or function (requires `cacheComponents: true`)
- `'use cache: private'` — cache with access to runtime APIs, browser-only (experimental)
- `'use cache: remote'` — cache in a remote handler (Redis/KV) shared across server instances

---

## `'use client'`

Marks a file as a **Client Component entry point**. Required for hooks, event handlers, and browser APIs.

```tsx
'use client'
import { useState } from 'react'

export default function Counter() {
  const [count, setCount] = useState(0)
  return <button onClick={() => setCount(count + 1)}>{count}</button>
}
```

### Rules
- Place at the **top of the file**, before any imports
- Only add it at the **boundary** — all children are automatically client-side; you don't need it on every file
- Props passed from Server → Client must be **serializable** (no functions, no class instances, no Symbols)

```tsx
// ❌ Invalid — function props are not serializable across the boundary
export default function Counter({ onClick }) { ... }
```

### Composition pattern
```tsx
// page.tsx (Server Component)
import Header from './header'      // Server Component
import Counter from './counter'    // Client Component ('use client' inside)

export default function Page() {
  return <div><Header /><Counter /></div>
}
```

> See [React docs](https://react.dev/reference/rsc/use-client) for full reference.

---

## `'use server'`

Marks a function or file as a **Server Action** (Server Function). The function executes on the server and its result is serialized back to the client.

### File-level (all exports are Server Actions)
```tsx
// app/actions.ts
'use server'
import { db } from '@/lib/db'
import { auth } from '@/lib/auth'

export async function createUser(data: { name: string; email: string }) {
  const session = await auth()
  if (!session?.user) throw new Error('Unauthorized')
  const user = await db.user.create({ data })
  return { id: user.id, name: user.name } // only return what UI needs
}
```

### Inline (inside a Server Component)
```tsx
export default async function PostPage({ params }) {
  const post = await getPost(params.id)

  async function updatePost(formData: FormData) {
    'use server'
    await savePost(params.id, formData)
    revalidatePath(`/posts/${params.id}`)
  }

  return <EditPost updatePostAction={updatePost} post={post} />
}
```

### Using Server Functions in Client Components
Import from a dedicated `'use server'` file — cannot define inline inside a Client Component:

```tsx
// app/components/my-button.tsx
'use client'
import { fetchUsers } from '../actions'

export default function MyButton() {
  return <button onClick={() => fetchUsers()}>Fetch Users</button>
}
```

### Security rules
- Always **authenticate and authorize** before sensitive operations — read auth from cookies/headers, never accept tokens as parameters
- **Constrain return values** — only return what the UI needs, never raw DB records
- Validate all inputs; treat Server Actions as public API endpoints

> See [React docs](https://react.dev/reference/rsc/use-server) for full reference.

---

## `'use cache'`

Caches the return value of a route, component, or async function. Requires `cacheComponents: true` in `next.config.ts`.

```ts
// next.config.ts
const nextConfig: NextConfig = { cacheComponents: true }
```

### Three scopes

```tsx
// File level — all exports must be async
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

### Cache keys
Generated from: **Build ID** + **Function ID** (hash of location/signature) + **serializable arguments** (including closed-over variables).

```tsx
// userId (closure) + filter (argument) = both part of cache key
async function Component({ userId }) {
  const getData = async (filter: string) => {
    'use cache'
    return fetch(`/api/users/${userId}/data?filter=${filter}`)
  }
  return getData('active')
}
```

### Serialization rules

| | Supported |
|---|---|
| **Arguments** | Primitives, plain objects, arrays, Dates, Maps, Sets, TypedArrays, ArrayBuffers, React elements (pass-through only) |
| **Return values** | Same as arguments, plus JSX |
| **Not supported** | Class instances, functions (except pass-through), Symbols, WeakMaps, WeakSets, URL instances |

**Pass-through pattern** — non-serializable values (children, Server Actions) can be passed through without introspecting:
```tsx
async function CachedWrapper({ children }: { children: ReactNode }) {
  'use cache'
  return <div><header>Cached</header>{children}</div>
  // children is passed through, not introspected — doesn't affect cache key
}
```

### Constraints
- **Cannot** call `cookies()`, `headers()`, `searchParams` inside — read outside and pass as args
- `React.cache` is isolated inside `'use cache'` boundaries (can't share values via `React.cache` across the boundary)

### Revalidation

**Default profile** (`'default'`): stale 5 min (client), revalidate 15 min (server), never expires.

```tsx
import { cacheLife, cacheTag, revalidateTag } from 'next/cache'

async function getProducts() {
  'use cache'
  cacheTag('products')       // tag for on-demand invalidation
  cacheLife('hours')         // built-in profile: stale 1h, revalidate 1h, expire 1d
  return fetch('/api/products').then(r => r.json())
}

// Server Action to invalidate:
export async function updateProduct() {
  'use server'
  await db.products.update(...)
  revalidateTag('products')
}
```

**Built-in `cacheLife` profiles**: `seconds`, `minutes`, `hours`, `days`, `weeks`, `max`

**Client minimum**: router enforces 30-second minimum stale time regardless of config.

### Runtime caching behavior

| Environment | Behavior |
|---|---|
| Serverless | In-memory; entries typically don't persist across requests |
| Self-hosted | In-memory; entries persist across requests |

Use `'use cache: remote'` when in-memory isn't enough.

### Caching whole routes
Add `'use cache'` to both `layout.tsx` and `page.tsx` — each is a separate cache entry:

```tsx
// app/layout.tsx
'use cache'
export default async function Layout({ children }) {
  return <div>{children}</div>
}

// app/page.tsx
'use cache'
export default async function Page() { ... }
```

### Troubleshooting: build hang (cache timeout)
If build hangs with "Filling a cache during prerender timed out" — you're passing Promises that resolve to runtime data (params, searchParams, cookies) into a `'use cache'` scope. Await them outside and pass the values in.

```bash
# Verbose cache logging
NEXT_PRIVATE_DEBUG_CACHE=1 npm run dev
```

### Platform support
Supported: Node.js server, Docker. **Not supported**: Static export.

---

## `'use cache: private'` ⚠️ Experimental

Like `'use cache'` but allows `cookies()`, `headers()`, and `searchParams` inside. Results are cached **only in the browser's memory** — never stored on the server, never shared across users.

Use when:
- The function already accesses runtime data and refactoring to pass values as args isn't practical
- Compliance requirements prevent storing data server-side

**Not available** in Route Handlers.

```tsx
async function getRecommendations(productId: string) {
  'use cache: private'
  cacheTag(`recommendations-${productId}`)
  cacheLife({ stale: 60 }) // stale must be ≥ 30s for runtime prefetching

  const sessionId = (await cookies()).get('session-id')?.value || 'guest'
  return getPersonalizedRecs(productId, sessionId)
}
```

### API access comparison

| API | `use cache` | `'use cache: private'` |
|---|---|---|
| `cookies()` | ❌ | ✅ |
| `headers()` | ❌ | ✅ |
| `searchParams` | ❌ | ✅ |
| `connection()` | ❌ | ❌ |

### Nesting rules
- Remote caches **cannot** nest inside private caches
- Private caches **cannot** nest inside remote caches

---

## `'use cache: remote'`

Stores cache entries in a **remote handler** (Redis, KV, etc.) shared across all server instances. Provides durable caching with cross-instance hit rates. Tradeoffs: infrastructure cost + network latency on lookups.

Configure the handler via `cacheHandlers` in `next.config.ts`.

```tsx
async function getProductPrice(productId: string, currency: string) {
  'use cache: remote'
  cacheTag(`price-${productId}`)
  cacheLife({ expire: 3600 }) // 1 hour
  return db.products.getPrice(productId, currency)
}
```

### When to use remote caching
✅ Rate-limited APIs, slow backends, expensive computations, flaky services, serverless where memory isn't shared across instances.

❌ Already fast ops (<50ms), highly unique cache keys per request, frequently changing data (seconds to minutes).

### Cache key strategy — fewer unique values = better hit rate
```tsx
// ✅ Cache on category (low cardinality), filter in memory
async function getProductsByCategory(category: string) {
  'use cache: remote'
  return db.products.findByCategory(category)
}
// Then filter by price in the component, not in the cached function

// ✅ Cache on language (low cardinality), not user session
async function getCMSContent(language: string) {
  'use cache: remote'
  return cms.getHomeContent(language)
}
// Extract language from cookie outside and pass it in
```

### Three-directive comparison

| Feature | `use cache` | `'use cache: remote'` | `'use cache: private'` |
|---|---|---|---|
| Server-side storage | In-memory | Remote handler | None |
| Scope | Shared (all users) | Shared (all users) | Per-client (browser) |
| `cookies()`/`headers()` | ❌ (pass as args) | ❌ (pass as args) | ✅ |
| Serverless hit rate outside static shell | Low | High | N/A |
| Extra cost | None | Infrastructure | None |

### Nesting rules
- Remote inside remote ✅
- Remote inside regular cache ✅
- Remote inside private ❌
- Private inside remote ❌

### Deferred execution pattern (with `connection()`)
```tsx
async function DashboardStats() {
  await connection() // explicitly defer to request time
  const stats = await getGlobalStats()
  return <StatsDisplay stats={stats} />
}

async function getGlobalStats() {
  'use cache: remote'
  cacheTag('global-stats')
  cacheLife({ expire: 60 }) // 1 min — DB sees ≤1 req/min regardless of traffic
  return db.analytics.aggregate({ ... })
}
```

### Mixed caching strategy pattern
```tsx
// Static shell — build time
async function getProduct(id) { 'use cache'; ... }

// Shared runtime — remote handler
async function getProductPrice(id) { 'use cache: remote'; ... }

// Per-user — browser only
async function getRecommendations(productId) { 'use cache: private'; ... }
```

### Platform support
Supported: Node.js server, Docker, Adapters. **Not supported**: Static export.