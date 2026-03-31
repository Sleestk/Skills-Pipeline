# Memory Usage

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/memory-usage

## Quick Wins

### `experimental.webpackMemoryOptimizations` (v15+)
```js
// next.config.js
module.exports = { experimental: { webpackMemoryOptimizations: true } }
```
Reduces max memory at slight cost to compilation time.

### Webpack build worker (default since v14.1)
Runs Webpack in a separate Node.js worker. Set `experimental.webpackBuildWorker: true` if you have a custom Webpack config and v14.1+.

### Disable Webpack cache (reduces memory, slower rebuilds)
```js
webpack: (config, { dev }) => {
  if (config.cache && !dev) {
    config.cache = Object.freeze({ type: 'memory' })
  }
  return config
}
```

### Disable static analysis (TypeScript) for builds
```js
module.exports = { typescript: { ignoreBuildErrors: true } }
```
⚠️ Only use if TypeScript errors are checked separately in CI.

### Disable source maps
```js
module.exports = {
  productionBrowserSourceMaps: false,
  experimental: { serverSourceMaps: false, enablePrerenderSourceMaps: false }
}
```

### Disable entry preloading
```js
module.exports = { experimental: { preloadEntriesOnStart: false } }
```
Trades faster response times for lower initial memory footprint.

## Profiling Tools

### `--experimental-debug-memory-usage` (v14.2+)
```bash
next build --experimental-debug-memory-usage
```
Prints heap usage + GC stats during build. Takes heap snapshots automatically near the memory limit.

### Heap profile
```bash
node --heap-prof node_modules/next/dist/bin/next build
# Creates .heapprofile → load in Chrome DevTools Memory tab
```

### Heap snapshot with inspector
```bash
NODE_OPTIONS=--inspect next build
# or: NODE_OPTIONS=--inspect-brk next dev
# Connect Chrome DevTools → chrome://inspect
# Send SIGUSR2 to process to take snapshot
```

### Reduce dependencies
Use [Bundle Analyzer](/docs/app/guides/package-bundling) to find large deps.

### Edge memory issues
Fixed in Next.js v14.1.3 — update if using Edge runtime.