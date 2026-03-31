# Next.js Configuration Reference (`next.config.ts`)

> Version: 16.2.1 | Source: https://nextjs.org/docs/app/api-reference/config

---

## File Format

Prefer `next.config.ts` for TypeScript projects:

```ts
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  /* config options here */
}

export default nextConfig
```

Also supported: `next.config.js` (CommonJS), `next.config.mjs` (ESM).  
**Not supported**: `.cjs` or `.cts` extensions.

### Function form (phase-aware)

```ts
import { PHASE_DEVELOPMENT_SERVER } from 'next/constants'

export default (phase, { defaultConfig }) => {
  if (phase === PHASE_DEVELOPMENT_SERVER) {
    return { /* dev-only options */ }
  }
  return { /* all other phases */ }
}
```

Async function form is also supported since Next.js 12.1.

### Node.js Native TypeScript Resolver (`next.config.ts`)

Available on Node.js v22.10.0+. Enables ESM syntax (top-level `await`, dynamic `import()`) in `next.config.ts`.  
For CommonJS projects, prefer `next.config.mts` to avoid ambiguity. Enabled by default on v22.18.0+; for v22.10–22.17.x, opt in via `NODE_OPTIONS=--experimental-transform-types`.

---

## Unit Testing (`next.config.js`)

```ts
import { unstable_getResponseFromNextConfig } from 'next/experimental/testing/server'

const response = await unstable_getResponseFromNextConfig({
  url: 'https://example.com/test',
  nextConfig: {
    async redirects() {
      return [{ source: '/test', destination: '/test2', permanent: false }]
    },
  },
})
// response.status === 307
```

> Only considers `next.config.js` fields — not proxy or filesystem routes.

---

## All Config Options (A–Z)

| Option | Description |
|---|---|
| `adapterPath` | Custom adapter to hook into the build process |
| `allowedDevOrigins` | Additional origins that can request the dev server |
| `appDir` | Enable App Router (layouts, streaming, etc.) |
| `assetPrefix` | CDN prefix for static assets |
| `authInterrupts` | Enable `forbidden()` / `unauthorized()` APIs (experimental) |
| `basePath` | Deploy under a sub-path of a domain |
| `cacheComponents` | Enable `'use cache'` directive (v16) |
| `cacheHandlers` | Custom cache handlers for `'use cache'` |
| `cacheLife` | Custom `cacheLife` profiles |
| `compress` | Enable gzip compression (server only) |
| `crossOrigin` | `crossorigin` attribute on `<script>` tags |
| `cssChunking` | Control CSS file chunking strategy |
| `deploymentId` | Deployment identifier for version skew protection |
| `devIndicators` | On-screen dev route indicator config |
| `distDir` | Custom build output directory (default: `.next`) |
| `env` | Build-time environment variables |
| `expireTime` | Stale-while-revalidate expire time for ISR pages |
| `exportPathMap` | Custom page map for `next export` (Pages Router) |
| `generateBuildId` | Custom build ID for cache busting |
| `generateEtags` | Toggle ETag generation (default: `true`) |
| `headers` | Custom HTTP response headers |
| `htmlLimitedBots` | User agents that receive blocking metadata |
| `httpAgentOptions` | HTTP Keep-Alive config |
| `images` | `next/image` loader and domain config |
| `cacheHandler` (`incrementalCacheHandlerPath`) | External cache store (Redis, Memcached, etc.) |
| `inlineCss` | Enable inline CSS support |
| `logging` | Dev-mode terminal logging (fetch, requests, browser console) |
| `mdxRs` | Use Rust compiler for MDX files |
| `onDemandEntries` | In-memory page disposal settings for dev |
| `optimizePackageImports` | Tree-shake specific package imports |
| `output` | Build output mode (`'standalone'`, `'export'`) |
| `pageExtensions` | Custom file extensions for Pages Router pages |
| `poweredByHeader` | Toggle `x-powered-by` header (default: `true`) |
| `productionBrowserSourceMaps` | Emit source maps in production builds |
| `proxyClientMaxBodySize` | Max request body size for proxy |
| `reactCompiler` | Enable React Compiler for automatic memoization |
| `reactMaxHeadersLength` | Max length of React-emitted response headers |
| `reactStrictMode` | Enable React Strict Mode |
| `redirects` | URL redirects |
| `rewrites` | URL rewrites |
| `sassOptions` | Sass compiler options |
| `serverActions` | Server Actions behavior (allowed origins, body size limit) |
| `serverComponentsHmrCache` | Cache `fetch` responses in Server Components across HMR |
| `serverExternalPackages` | Opt packages out of Server Component bundling (use native `require`) |
| `staleTimes` | Client-side router cache invalidation time |
| `staticGeneration*` | Static generation concurrency, retries, timeout |
| `taint` | Enable React taint API for objects/values |
| `trailingSlash` | Normalize URLs with/without trailing slash |
| `transpilePackages` | Auto-transpile local packages or `node_modules` deps |
| `turbopack` | Turbopack-specific bundler options |
| `turbopackFileSystemCache` | Enable filesystem caching for Turbopack builds |
| `turbopack.ignoreIssue` | Suppress specific Turbopack errors/warnings |
| `typedRoutes` | Statically typed `next/link` hrefs |
| `typescript` | TypeScript error handling and custom `tsconfig` path |
| `urlImports` | Allow importing modules from external URLs |
| `useLightningcss` | Use Lightning CSS instead of PostCSS (experimental) |
| `viewTransition` | Enable React View Transitions API in App Router |
| `webpack` | Custom Webpack configuration |
| `webVitalsAttribution` | Attribute Web Vitals issues to specific components |

---

## Key Options — Detail

### `typescript`

```ts
const nextConfig: NextConfig = {
  typescript: {
    ignoreBuildErrors: true,      // ⚠️ skip type errors in production build
    tsconfigPath: 'tsconfig.build.json', // custom tsconfig for builds
  },
}
```

- `ignoreBuildErrors` — Dangerous. Use only if type checks run separately in CI.
- `tsconfigPath` — Lets you use a relaxed config for builds while keeping strict IDE checks.

### `typedRoutes`

```ts
const nextConfig: NextConfig = { typedRoutes: true }
```

- Generates `.d.ts` link types during `next dev` / `next build` / `next typegen`
- Validates `href` in `next/link`, `useRouter().push/replace/prefetch`
- Non-literal strings need `as Route` cast: `router.push(('/blog/' + slug) as Route)`
- Use `Route<T>` generic for custom link-wrapping components

### `images`

Configure allowed remote domains, loader, formats, sizes — required for any external image domain.

### `headers` / `redirects` / `rewrites`

Async functions returning arrays of rules. Testable via `unstable_getResponseFromNextConfig`.

### `serverActions`

```ts
serverActions: {
  allowedOrigins: ['my-proxy.com', '*.my-proxy.com'],
  bodySizeLimit: '2mb',
}
```

### `output`

```ts
output: 'standalone'  // self-contained Node.js server (Docker-friendly)
output: 'export'      // static HTML export (no server required)
```

### `cacheComponents` (v16)

```ts
const nextConfig: NextConfig = { cacheComponents: true }
```

Enables the `'use cache'` directive. When enabled, `'use cache'` replaces the `fetch()` cache options model. See `'use cache'` in SKILL.md Step 2.

---

## TypeScript Config Deep Dive

### `next-env.d.ts`

Auto-generated — do NOT edit. Add to `.gitignore`. Must be in `tsconfig.json` `include` array.

For custom type declarations, create a separate `new-types.d.ts` and reference it in `tsconfig.json`.

### `typedEnv` (experimental)

```ts
experimental: { typedEnv: true }
```

Generates IntelliSense for environment variables. Run with `NODE_ENV=production next dev` to include production vars.

### Route-Aware Type Helpers

Available globally (no import needed), generated by `next dev` / `next build` / `next typegen`:
- `PageProps` — type for `page.tsx` props
- `LayoutProps` — type for `layout.tsx` props
- `RouteContext` — type for `route.ts` context

### End-to-End Type Safety (App Router)

- Server Components can return `Date`, `Map`, `Set`, etc. — no manual serialization
- Data fetching colocated in components removes the old `_app` typing friction

---

## ESLint Config (v16)

> `next lint` was **removed in v16**. Use the ESLint CLI directly.

### Setup (flat config)

```bash
npm i -D eslint eslint-config-next
```

```js
// eslint.config.mjs
import { defineConfig, globalIgnores } from 'eslint/config'
import nextVitals from 'eslint-config-next/core-web-vitals'

export default defineConfig([
  ...nextVitals,
  globalIgnores(['.next/**', 'out/**', 'build/**', 'next-env.d.ts']),
])
```

### Config variants

| Config | Use case |
|---|---|
| `eslint-config-next` | Base (JS + TS, Next.js + React + Hooks rules) |
| `eslint-config-next/core-web-vitals` | Base + CWV rules promoted to errors (recommended) |
| `eslint-config-next/typescript` | Adds `typescript-eslint` rules on top |

### With Prettier

```bash
npm i -D eslint-config-prettier
```

```js
import prettier from 'eslint-config-prettier/flat'
export default defineConfig([...nextVitals, prettier, globalIgnores([...])])
```

### Monorepo — specifying root dir

```js
settings: { next: { rootDir: 'packages/my-app/' } }
```

`rootDir` accepts path, glob, or array.

### Lint staged files

```js
// .lintstagedrc.js
const buildEslintCommand = (filenames) =>
  `eslint --fix ${filenames.map((f) => `"${path.relative(process.cwd(), f)}"`).join(' ')}`

module.exports = { '*.{js,jsx,ts,tsx}': [buildEslintCommand] }
```

### `next lint` migration codemod (v16)

```bash
npx @next/codemod@canary migrate-from-next-lint .
```