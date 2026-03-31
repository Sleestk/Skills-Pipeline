# Single-Page Applications (SPAs)

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/single-page-applications

Next.js fully supports SPAs and lets you progressively adopt server features as needed.

## SPA with `use` API + Context Provider

Start fetch in Server Component, pass unawaited promise to context:

```tsx
// app/layout.tsx
import { UserProvider } from './user-provider'
import { getUser } from './user'
export default function RootLayout({ children }) {
  const userPromise = getUser()  // NOT awaited
  return <html><body><UserProvider userPromise={userPromise}>{children}</UserProvider></body></html>
}
```

```ts
// app/user-provider.tsx
'use client'
import { createContext, useContext } from 'react'
const UserContext = createContext(null)
export function useUser() { return useContext(UserContext) }
export function UserProvider({ children, userPromise }) {
  return <UserContext.Provider value={{ userPromise }}>{children}</UserContext.Provider>
}
```

```tsx
// app/profile.tsx
'use client'
import { use } from 'react'
import { useUser } from './user-provider'
export function Profile() {
  const { userPromise } = useUser()
  const user = use(userPromise)  // suspends here
  return '...'
}
```

## SPA with SWR (SWR 2.3.0+ / React 19+)

```tsx
// app/layout.tsx — Server Component
import { SWRConfig } from 'swr'
import { getUser } from './user'
export default function RootLayout({ children }) {
  return (
    <SWRConfig value={{ fallback: { '/api/user': getUser() } }}>
      {children}
    </SWRConfig>
  )
}

// app/profile.tsx — Client Component (unchanged SWR code)
'use client'
import useSWR from 'swr'
export function Profile() {
  const { data } = useSWR('/api/user', fetcher)
  return '...'
}
```

## Client-Only Components

```tsx
import dynamic from 'next/dynamic'
const ClientOnlyComponent = dynamic(() => import('./component'), { ssr: false })
```

## Shallow Routing (SPA-style URL updates)

```tsx
'use client'
import { useSearchParams } from 'next/navigation'
export default function SortProducts() {
  const searchParams = useSearchParams()
  function updateSorting(sortOrder: string) {
    const params = new URLSearchParams(searchParams.toString())
    params.set('sort', sortOrder)
    window.history.pushState(null, '', `?${params.toString()}`)
  }
  return <button onClick={() => updateSorting('asc')}>Sort</button>
}
```

`pushState`/`replaceState` integrate with `usePathname` and `useSearchParams`.

## Server Actions in Client Components

```tsx
'use client'
import { create } from './actions'
export function Button() {
  return <button onClick={() => create()}>Create</button>
}
```

## Static Export (optional)

```ts
// next.config.ts
const nextConfig: NextConfig = { output: 'export' }
```

Generates per-route HTML files (better than single `index.html`). Note: server features not supported with static export.

## Migration guides
- [From Create React App](/docs/app/guides/migrating/from-create-react-app)
- [From Vite](/docs/app/guides/migrating/from-vite)