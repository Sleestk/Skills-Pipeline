# Migrating from Create React App

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/migrating/from-create-react-app

Goal: working Next.js app as SPA first, then adopt server features incrementally.

## Steps

### 1. Install Next.js
```bash
npm install next@latest
```

### 2. `next.config.ts`
```ts
import type { NextConfig } from 'next'
const nextConfig: NextConfig = {
  output: 'export',   // SPA mode
  distDir: 'build',   // match CRA output dir
}
export default nextConfig
```

### 3. Root Layout (`app/layout.tsx`)
Port `public/index.html` — replace `<body><div id="root"></div></body>` with `<div id="root">{children}</div>`. Use Next.js Metadata API for `<title>` and `<meta>`:
```tsx
import type { Metadata } from 'next'
export const metadata: Metadata = { title: 'React App', description: '...' }
export default function RootLayout({ children }: { children: React.ReactNode }) {
  return <html lang="en"><body><div id="root">{children}</div></body></html>
}
```

### 4. Catch-all entry point (`app/[[...slug]]/page.tsx`)
```tsx
export function generateStaticParams() { return [{ slug: [''] }] }
export default function Page() { return '...' }
```

### 5. Client-only component (`app/[[...slug]]/client.tsx`)
```tsx
'use client'
import dynamic from 'next/dynamic'
const App = dynamic(() => import('../../App'), { ssr: false })
export function ClientOnly() { return <App /> }
```
Update `page.tsx` to use `<ClientOnly />`.

### 6. Static image imports
```tsx
// Before: import logo from '/logo.png'
// After:
import logo from '../public/logo.png'
<img src={logo.src} />
```

### 7. Environment variables
Change `REACT_APP_` prefix to `NEXT_PUBLIC_`.

### 8. `package.json` scripts
```json
{ "scripts": { "dev": "next dev", "build": "next build", "start": "npx serve@latest ./build" } }
```
Add `.next` and `next-env.d.ts` to `.gitignore`.

### 9. Clean up
Remove: `public/index.html`, `src/index.tsx`, `src/react-app-env.d.ts`, `reportWebVitals` setup, `react-scripts` dependency.

## Additional Considerations

- **Custom `homepage` field** → use [`basePath`](https://nextjs.org/docs/app/api-reference/config/next-config-js/basePath)
- **Service worker** → see [PWA guide](/docs/app/guides/progressive-web-apps)
- **`proxy` field in package.json** → use Next.js `rewrites` in `next.config.ts`
- **Custom webpack/Babel** → extend `next.config.ts` webpack option (requires `--webpack` flag)
- **TypeScript** → add `next-env.d.ts` to `tsconfig.json` `include` array

## Remove `output: 'export'` when ready
Removing it enables SSR, Server Components, ISR, Server Actions, and all other server features.