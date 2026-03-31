# Lazy Loading

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/lazy-loading

Defer loading of Client Components and libraries until needed. Server Components are auto code-split. Lazy loading applies to Client Components only.

## `next/dynamic` (composite of `React.lazy` + Suspense)

```jsx
// app/page.js
'use client'
import { useState } from 'react'
import dynamic from 'next/dynamic'

const ComponentA = dynamic(() => import('../components/A'))                    // loads in separate bundle immediately
const ComponentB = dynamic(() => import('../components/B'))                    // load on demand
const ComponentC = dynamic(() => import('../components/C'), { ssr: false })   // client-only, no SSR
```

## Skip SSR

```jsx
const ComponentC = dynamic(() => import('../components/C'), { ssr: false })
```

> `ssr: false` only works in Client Components.

## Dynamic import of Server Components

```jsx
import dynamic from 'next/dynamic'
const ServerComponent = dynamic(() => import('../components/ServerComponent'))
// Only Client Component children are lazy-loaded, not the Server Component itself
```

## External Libraries on Demand

```jsx
'use client'
import { useState } from 'react'

export default function Page() {
  const [results, setResults] = useState()
  return (
    <input
      type="text"
      onChange={async (e) => {
        const Fuse = (await import('fuse.js')).default
        const fuse = new Fuse(['Tim', 'Joe', 'Bel'])
        setResults(fuse.search(e.currentTarget.value))
      }}
    />
  )
}
```

## Custom Loading Component

```jsx
const WithCustomLoading = dynamic(
  () => import('../components/WithCustomLoading'),
  { loading: () => <p>Loading...</p> }
)
```

## Named Exports

```jsx
const ClientComponent = dynamic(() =>
  import('../components/hello').then((mod) => mod.Hello)
)
```

## Magic Comments (Bundler hints)

```js
// Skip bundling — import at runtime
const runtime = await import(/* webpackIgnore: true */ 'runtime-module')
const plugin = await import(/* turbopackIgnore: true */ pluginPath)

// Suppress build error for optional modules (Turbopack only)
const feature = await import(/* turbopackOptional: true */ './optional-feature')
```

> Magic comments only work with dynamic `import()`, not static `import` statements.
> `webpackOptional` is not supported; use `turbopackOptional` instead.