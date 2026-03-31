# Turbopack Reference (v16.2.1)

Turbopack is an **incremental, Rust-based bundler** built into Next.js. It is the **default bundler** as of v16 for both `dev` and `build`.

---

## Why Turbopack?

| Advantage | Detail |
|---|---|
| **Unified graph** | Single graph for all environments (client + server) — no stitching |
| **Bundled in dev** | Bundles rather than relying on native ESM — keeps large apps fast |
| **Incremental computation** | Caches down to the function level; parallelizes across cores |
| **Lazy bundling** | Only bundles what the dev server actually requests |

---

## Getting Started

No configuration needed — Turbopack is the default:

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build"
  }
}
```

To opt out and use Webpack instead:

```json
{
  "scripts": {
    "dev": "next dev --webpack",
    "build": "next build --webpack"
  }
}
```

---

## Supported Platforms

| Platform | Architecture |
|---|---|
| macOS (Darwin) | x64, ARM64 |
| Windows | x64, ARM64 |
| Linux (glibc) | x64, ARM64 |
| Linux (musl) | x64, ARM64 |

On unsupported platforms (FreeBSD, OpenBSD), Next.js falls back to WASM bindings — core SWC features work but **Turbopack is not supported**. Use `--webpack` on these platforms.

---

## Supported Features

### Language

| Feature | Status | Notes |
|---|---|---|
| JavaScript & TypeScript | ✅ | Uses SWC. No type-checking — run `tsc --watch` separately |
| ESNext | ✅ | Matches SWC's coverage |
| CommonJS (`require`) | ✅ | Handled out of the box |
| ESM (static + dynamic `import`) | ✅ | Fully supported |
| Babel | ✅ (v16+) | Auto-detected from config file. SWC always handles internal transforms. `node_modules` excluded unless `babel-loader` manually configured |

### Framework & React

| Feature | Status | Notes |
|---|---|---|
| JSX / TSX | ✅ | SWC compilation |
| Fast Refresh | ✅ | Zero config |
| React Server Components | ✅ | Correct server/client bundling for App Router |
| Auto root layout creation | ❌ | Must create `app/layout.tsx` manually |

### CSS & Styling

| Feature | Status | Notes |
|---|---|---|
| Global CSS | ✅ | Import `.css` directly |
| CSS Modules | ✅ | `.module.css` via Lightning CSS |
| CSS Nesting | ✅ | Modern CSS nesting via Lightning CSS |
| `@import` | ✅ | Combine CSS files |
| PostCSS | ✅ | Auto-processes `postcss.config.*` in Node.js worker pool |
| Sass / SCSS | ✅ | Supported in Next.js. `sassOptions.functions` not supported (see below) |
| Less | 🔜 | Not default; will need loader config |
| Lightning CSS | ✅ (in use) | Some legacy CSS Modules features unsupported |

### Assets & Module Resolution

| Feature | Status | Notes |
|---|---|---|
| Static assets (images, fonts) | ✅ | `import img from './img.png'` works; returns object for `<Image />` |
| JSON imports | ✅ | Named or default imports |
| Path aliases (`tsconfig.json` `paths`) | ✅ | Reads `baseUrl` + `paths` |
| Manual aliases (`resolveAlias`) | ✅ | Configure in `next.config.js` |
| Custom extensions (`resolveExtensions`) | ✅ | Configure in `next.config.js` |
| AMD | ⚠️ | Basic transforms only |

### Magic Comments (dynamic imports)

Works with `import()`, `require()`, `require.resolve()`, `new Worker()` — **not** static `import`.

| Comment | Webpack | Turbopack | Description |
|---|---|---|---|
| `webpackIgnore: true` | ✓ | ✓ | Skip bundling, preserve import |
| `turbopackIgnore: true` | ✗ | ✓ | Skip bundling (Turbopack-only) |
| `turbopackOptional: true` | ✗ | ✓ | Suppress resolve errors |

---

## Known Gaps vs Webpack

### Filesystem root
Turbopack resolves from the project root. Files outside it (e.g. `npm link` / `pnpm link` symlinks) won't resolve by default. Fix:

```js
// next.config.js
module.exports = {
  turbopack: { root: '/path/to/monorepo/root' }
}
```

### CSS Module ordering
Turbopack follows JS import order for CSS Modules. Webpack sometimes ignores this for side-effect-free files. If you see rendering changes, explicitly enforce order:

```scss
/* button.module.css */
@import './utils.module.css';  /* force ordering */
```

### Sass `node_modules` imports
Turbopack does **not** support the legacy tilde `~` syntax:

```scss
/* ❌ Webpack-only */
@import '~bootstrap/dist/css/bootstrap.min.css';

/* ✅ Turbopack */
@import 'bootstrap/dist/css/bootstrap.min.css';
```

Or map it via config:

```js
module.exports = {
  turbopack: { resolveAlias: { '~*': '*' } }
}
```

### Build caching
Turbopack's filesystem cache is in beta (v16+):

```js
// next.config.js
module.exports = {
  experimental: {
    turbopackFileSystemCacheForDev: true,   // default true
    turbopackFileSystemCacheForBuild: true,  // default false, opt-in
  }
}
```

> When benchmarking Turbopack vs Webpack, delete `.next/` between runs for fair cold comparison.

### Webpack plugins
**Not supported.** Use `turbopack.rules` (webpack loaders) as an alternative. If you need webpack plugins specifically, stay on `--webpack`.

---

## Unsupported / Not Planned

- **Legacy CSS Modules**: standalone `:local`/`:global`, `@value` rule, `:import`/`:export` ICSS, `composes` from `.css` (must use `.module.css`), `@import` inside CSS Modules of `.css` files
- **`sassOptions.functions`** — JS functions can't run in Turbopack's Rust architecture
- **`webpack()` in `next.config.js`** — use `turbopack` key instead
- **Yarn PnP** — not planned
- **`experimental.urlImports`** — not planned
- **`experimental.esmExternals`** — not planned
- **`experimental.nextScriptWorkers`** — future
- **`experimental.fallbackNodePolyfills`** — future

---

## Configuration (`next.config.js`)

```js
module.exports = {
  turbopack: {
    resolveAlias: {
      underscore: 'lodash',
    },
    resolveExtensions: ['.mdx', '.tsx', '.ts', '.jsx', '.js', '.json'],
    rules: {
      // webpack loader config
    },
  },
}
```

### Experimental options

| Option | Dev default | Build default | Description |
|---|---|---|---|
| `turbopackFileSystemCacheForDev` | `true` | N/A | FS cache for dev server |
| `turbopackFileSystemCacheForBuild` | N/A | `false` | FS cache for builds (opt-in) |
| `turbopackMinify` | `false` | `true` | Minification |
| `turbopackSourceMaps` | `true` | `productionBrowserSourceMaps` | Source maps |
| `turbopackInputSourceMaps` | `true` | `true` | Extract source maps from inputs |
| `turbopackTreeShaking` | `false` | `false` | Advanced module-fragments tree shaking |
| `turbopackRemoveUnusedImports` | `false` | `true` | Remove unused imports (requires `RemoveUnusedExports`) |
| `turbopackRemoveUnusedExports` | `false` | `true` | Remove unused exports |
| `turbopackInferModuleSideEffects` | `true` | `true` | Local analysis for side-effect-free modules |
| `turbopackScopeHoisting` | `false` | `true` | Scope hoisting (always off in dev) |
| `turbopackClientSideNestedAsyncChunking` | `false` | `true` | Nested async chunking (client) |
| `turbopackServerSideNestedAsyncChunking` | `false` | `false` | Nested async chunking (server) |
| `turbopackImportTypeBytes` | `false` | `false` | `with {type: "bytes"}` ESM imports |
| `turbopackUseBuiltinBabel` | `true` | `true` | Auto Babel loader from config file |
| `turbopackUseBuiltinSass` | `true` | `true` | Auto Sass loader |
| `turbopackModuleIds` | `'named'` | `'deterministic'` | Module ID strategy |

---

## Performance Debugging

Generate a trace file for the Next.js team:

```bash
NEXT_TURBOPACK_TRACING=1 next dev
# outputs: .next/dev/trace-turbopack
```

Attach to a GitHub issue at https://github.com/vercel/next.js.

---

## Version History

| Version | Change |
|---|---|
| v16.0.0 | Default bundler for Next.js; auto Babel support from config file |
| v15.5.0 | Build support (beta) |
| v15.3.0 | Experimental build support |
| v15.0.0 | Dev mode stable |