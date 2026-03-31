# Migrating from Vite

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/migrating/from-vite

## Steps

### 1. Install Next.js
```bash
npm install next@latest
```

### 2. `next.config.mjs`
```js
const nextConfig = {
  output: 'export',   // SPA mode
  distDir: './dist',
}
export default nextConfig
```

### 3. Update `tsconfig.json`
Remove `tsconfig.node.json` project reference. Add to `include`: `./dist/types/**/*.ts`, `./next-env.d.ts`. Add to `exclude`: `./node_modules`. Add to `plugins`: `{ "name": "next" }`. Set: `esModuleInterop: true`, `jsx: "react-jsx"`, `allowJs: true`, `forceConsistentCasingInFileNames: true`, `incremental: true`.

### 4. Root Layout (`app/layout.tsx`)
Convert `index.html` — replace `<div id="root"></div>` + `<script>` with `<div id="root">{children}</div>`. Remove `charset` and `viewport` (Next.js adds them). Use Metadata API for title/description.

### 5. Entry point (`app/[[...slug]]/page.tsx`)
```tsx
import '../../index.css'
export function generateStaticParams() { return [{ slug: [''] }] }
export default function Page() { return '...' }
```

### 6. Client-only component (`app/[[...slug]]/client.tsx`)
```tsx
'use client'
import dynamic from 'next/dynamic'
const App = dynamic(() => import('../../App'), { ssr: false })
export function ClientOnly() { return <App /> }
```
Update `page.tsx` to import and render `<ClientOnly />`.

### 7. Static image imports
```tsx
// Before: import logo from '/logo.png' (returns URL string in Vite)
// After:
import logo from '../public/logo.png' // returns object
<img src={logo.src} />
```

### 8. Environment variables
- `VITE_` → `NEXT_PUBLIC_`
- `import.meta.env.MODE` → `process.env.NODE_ENV`
- `import.meta.env.PROD` → `process.env.NODE_ENV === 'production'`
- `import.meta.env.DEV` → `process.env.NODE_ENV !== 'production'`
- `import.meta.env.SSR` → `typeof window !== 'undefined'`
- For `BASE_URL`: add `NEXT_PUBLIC_BASE_PATH` to `.env` and set `basePath` in `next.config.mjs`

### 9. `package.json` scripts
```json
{ "scripts": { "dev": "next dev", "build": "next build", "start": "next start" } }
```
Add `.next`, `next-env.d.ts`, `dist` to `.gitignore`.

### 10. Clean up
Delete: `main.tsx`, `index.html`, `vite-env.d.ts`, `tsconfig.node.json`, `vite.config.ts`. Uninstall Vite deps.

## Next Steps After Migration

- Migrate from React Router to [Next.js App Router](/docs/app)
- Optimize images with [`<Image>`](/docs/app/api-reference/components/image)
- Optimize fonts with `next/font`
- Optimize scripts with `<Script>`
- Update ESLint for Next.js rules

Remove `output: 'export'` to unlock server features.