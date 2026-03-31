# CSS-in-JS

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/css-in-js

> **Warning:** CSS-in-JS with Server Components and Streaming requires library authors to support React concurrent rendering.

## Supported Libraries (Client Components in `app/`)
`ant-design`, `chakra-ui`, `@fluentui/react-components`, `kuma-ui`, `@mui/material`, `@mui/joy`, `pandacss`, `styled-jsx`, `styled-components`, `stylex`, `tamagui`, `tss-react`, `vanilla-extract`

Working on support: `emotion`

## Setup Pattern (3 steps)
1. **Style registry** to collect CSS rules during render
2. `useServerInsertedHTML` hook to inject rules before content
3. Client Component wrapping the app with the registry

### styled-jsx (v5.1.0+)
```tsx
// app/registry.tsx
'use client'
import { useState } from 'react'
import { useServerInsertedHTML } from 'next/navigation'
import { StyleRegistry, createStyleRegistry } from 'styled-jsx'

export default function StyledJsxRegistry({ children }: { children: React.ReactNode }) {
  const [jsxStyleRegistry] = useState(() => createStyleRegistry())
  useServerInsertedHTML(() => {
    const styles = jsxStyleRegistry.styles()
    jsxStyleRegistry.flush()
    return <>{styles}</>
  })
  return <StyleRegistry registry={jsxStyleRegistry}>{children}</StyleRegistry>
}
```

### styled-components (v6+)
```js
// next.config.js
module.exports = { compiler: { styledComponents: true } }
```

```tsx
// lib/registry.tsx
'use client'
import { useState } from 'react'
import { useServerInsertedHTML } from 'next/navigation'
import { ServerStyleSheet, StyleSheetManager } from 'styled-components'

export default function StyledComponentsRegistry({ children }: { children: React.ReactNode }) {
  const [styledComponentsStyleSheet] = useState(() => new ServerStyleSheet())
  useServerInsertedHTML(() => {
    const styles = styledComponentsStyleSheet.getStyleElement()
    styledComponentsStyleSheet.instance.clearTag()
    return <>{styles}</>
  })
  if (typeof window !== 'undefined') return <>{children}</>
  return <StyleSheetManager sheet={styledComponentsStyleSheet.instance}>{children}</StyleSheetManager>
}
```

### Wrap root layout with registry
```tsx
// app/layout.tsx
import Registry from './lib/registry'
export default function RootLayout({ children }) {
  return <html><body><Registry>{children}</Registry></body></html>
}
```