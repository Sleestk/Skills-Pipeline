# Fetching Data

> Next.js 16.2.1 — https://nextjs.org/docs/app/getting-started/fetching-data

## Server Components

### With the `fetch` API

Turn the component into an `async` function and `await` the `fetch` call:

```tsx
export default async function Page() {
  const data = await fetch('https://api.vercel.app/blog')
  const posts = await data.json()
  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  )
}
```

**Key notes:**
- Identical `fetch` requests in a React component tree are **memoized** by default — fetch data where you need it, no prop drilling required.
- `fetch` requests are **not cached by default** and block rendering until complete. Use `'use cache'` to cache results, or wrap in `<Suspense>` to stream.
- Log `fetch` calls in dev via the [`logging`](https://nextjs.org/docs/app/api-reference/config/next-config-js/logging) config option.

### With an ORM or database

Server Components run on the server — credentials and query logic are never exposed to the client:

```tsx
import { db, posts } from '@/lib/db'

export default async function Page() {
  const allPosts = await db.select().from(posts)
  return (
    <ul>
      {allPosts.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  )
}
```

Always authenticate and authorize requests. See the [data security guide](https://nextjs.org/docs/app/guides/data-security).

---

## Streaming

Break slow pages into chunks progressively sent from server to client.

### With `loading.js`

Add a `loading.tsx` file in the same folder as `page.tsx` to stream the **entire page**:

```tsx
// app/blog/loading.tsx
export default function Loading() {
  return <div>Loading...</div>
}
```

- On navigation, the user immediately sees the layout + loading state while the page renders.
- `loading.js` wraps `page.js` in a `<Suspense>` boundary automatically.
- A layout that accesses uncached/runtime data (`cookies()`, `headers()`, uncached fetches) does **not** fall back to `loading.js` — it blocks navigation until done. Fix: wrap in an explicit `<Suspense>` or move fetching to `page.js`.

### With `<Suspense>`

More granular control — stream only specific components:

```tsx
import { Suspense } from 'react'
import BlogList from '@/components/BlogList'
import BlogListSkeleton from '@/components/BlogListSkeleton'

export default function BlogPage() {
  return (
    <div>
      <header><h1>Welcome to the Blog</h1></header>
      <main>
        <Suspense fallback={<BlogListSkeleton />}>
          <BlogList />
        </Suspense>
      </main>
    </div>
  )
}
```

**Loading state best practices:** Use skeletons, spinners, or meaningful partial content (cover photo, title) — not just a plain spinner.

---

## Client Components

### With React's `use` API

Fetch in a Server Component, pass the **unawaited promise** to a Client Component:

```tsx
// app/blog/page.tsx (Server Component)
import Posts from '@/app/ui/posts'
import { Suspense } from 'react'

export default function Page() {
  const posts = getPosts() // Do NOT await

  return (
    <Suspense fallback={<div>Loading...</div>}>
      <Posts posts={posts} />
    </Suspense>
  )
}
```

```tsx
// app/ui/posts.tsx (Client Component)
'use client'
import { use } from 'react'

export default function Posts({ posts }: { posts: Promise<{ id: string; title: string }[]> }) {
  const allPosts = use(posts)
  return (
    <ul>
      {allPosts.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  )
}
```

### With community libraries (SWR / React Query)

```tsx
'use client'
import useSWR from 'swr'

const fetcher = (url) => fetch(url).then((r) => r.json())

export default function BlogPage() {
  const { data, error, isLoading } = useSWR('https://api.vercel.app/blog', fetcher)

  if (isLoading) return <div>Loading...</div>
  if (error) return <div>Error: {error.message}</div>

  return (
    <ul>
      {data.map((post: { id: string; title: string }) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  )
}
```

---

## Patterns

### Sequential data fetching

One request depends on another. Use `<Suspense>` to stream the dependent component independently:

```tsx
export default async function Page({ params }: { params: Promise<{ username: string }> }) {
  const { username } = await params
  const artist = await getArtist(username)

  return (
    <>
      <h1>{artist.name}</h1>
      <Suspense fallback={<div>Loading...</div>}>
        <Playlists artistID={artist.id} />
      </Suspense>
    </>
  )
}

async function Playlists({ artistID }: { artistID: string }) {
  const playlists = await getArtistPlaylists(artistID)
  return <ul>{playlists.map((p) => <li key={p.id}>{p.name}</li>)}</ul>
}
```

**Tip:** If the first request is slow and blocks everything, consider caching it if data changes infrequently.

### Parallel data fetching

Initiate requests before awaiting to avoid sequential blocking:

```tsx
export default async function Page({ params }: { params: Promise<{ username: string }> }) {
  const { username } = await params

  // Initiate both requests in parallel
  const artistData = getArtist(username)
  const albumsData = getAlbums(username)

  const [artist, albums] = await Promise.all([artistData, albumsData])

  return (
    <>
      <h1>{artist.name}</h1>
      <Albums list={albums} />
    </>
  )
}
```

> Use `Promise.allSettled` instead of `Promise.all` if you want partial failure handling (one failing request won't abort the rest).

### Sharing data with `React.cache` + Context

Create a cached fetch function:
```ts
// app/lib/user.ts
import { cache } from 'react'

export const getUser = cache(async () => {
  const res = await fetch('https://api.example.com/user')
  return res.json()
})
```

Pass the unawaited promise via a context provider in your root layout:
```tsx
// app/layout.tsx
import UserProvider from './user-provider'
import { getUser } from './lib/user'

export default function RootLayout({ children }: { children: React.ReactNode }) {
  const userPromise = getUser() // Don't await
  return (
    <html><body>
      <UserProvider userPromise={userPromise}>{children}</UserProvider>
    </body></html>
  )
}
```

Client Components resolve it with `use()` inside `<Suspense>`. Server Components call `getUser()` directly — `React.cache` deduplicates across both.

> `React.cache` is **request-scoped** — each request gets its own memoization scope, nothing is shared between requests.