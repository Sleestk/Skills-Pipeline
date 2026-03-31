# App Router Migration

> https://nextjs.org/docs/app/guides/migrating/app-router-migration

Incremental migration from Pages Router to App Router. Both routers can coexist — migrate route by route.

## Key differences

| | Pages Router | App Router |
|---|---|---|
| Data fetching | `getServerSideProps`, `getStaticProps`, `getStaticPaths` | `async` Server Components, `use cache`, Server Actions |
| Layouts | Custom `_app.js` / `_document.js` | `layout.tsx` file convention |
| API routes | `pages/api/` | Route Handlers (`route.ts`) |
| Styling | Same | Same |
| Images | Same | Same |
| Middleware | `middleware.ts` | `proxy.ts` (v16) |

## Migration approach

1. Create `app/` directory alongside `pages/`
2. Move root layout logic from `_app.js`/`_document.js` to `app/layout.tsx`
3. Migrate routes one at a time from `pages/` to `app/`
4. Convert data fetching:
   - `getServerSideProps` → `async` Server Component + `fetch`
   - `getStaticProps` → `async` Server Component + `'use cache'`
   - `getStaticPaths` → `generateStaticParams()`
5. Convert client-side state/effects → add `'use client'` where needed
6. Remove migrated pages from `pages/`

## `_app.js` → `app/layout.tsx`
Global CSS imports, context providers, and persistent UI move to the root layout.

## `_document.js` → `app/layout.tsx`
Custom `<html>`, `<head>`, `<body>` attributes move to the root layout.

## `getServerSideProps` → Server Component
```tsx
// Before (pages/blog/[slug].tsx)
export async function getServerSideProps({ params }) {
  const post = await fetchPost(params.slug)
  return { props: { post } }
}
export default function Page({ post }) { return <div>{post.title}</div> }

// After (app/blog/[slug]/page.tsx)
export default async function Page({ params }) {
  const { slug } = await params
  const post = await fetchPost(slug)
  return <div>{post.title}</div>
}
```

## `getStaticProps` + `getStaticPaths` → Server Component + `generateStaticParams`
```tsx
// app/blog/[slug]/page.tsx
export async function generateStaticParams() {
  const posts = await fetchPosts()
  return posts.map(p => ({ slug: p.slug }))
}
export default async function Page({ params }) {
  const { slug } = await params
  const post = await fetchPost(slug)
  return <div>{post.title}</div>
}
```

## `useRouter` → `useRouter` from `next/navigation` (different API)
- `router.push()`, `router.replace()`, `router.back()` still work
- `router.query` → `useSearchParams()` + `useParams()`
- `router.pathname` → `usePathname()`
- `router.asPath` → `usePathname()` + `useSearchParams()`

## See also
Full migration docs at https://nextjs.org/docs/app/guides/migrating/app-router-migration