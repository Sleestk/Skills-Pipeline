# Package Bundling

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/package-bundling

## Next.js Bundle Analyzer (Turbopack, experimental, v16.1+)

```bash
npx next experimental-analyze
# With output (for sharing/diffing):
npx next experimental-analyze --output
# Output saved to .next/diagnostics/analyze/
```

Opens interactive UI to filter by route/environment/type, trace import chains, see module sizes.

## `@next/bundle-analyzer` (Webpack)

```bash
npm install @next/bundle-analyzer
```

```js
// next.config.js
const withBundleAnalyzer = require('@next/bundle-analyzer')({ enabled: process.env.ANALYZE === 'true' })
module.exports = withBundleAnalyzer({})
```

```bash
ANALYZE=true npm run build
```

Opens 3 browser tabs with bundle visualizations.

## Fixing Large Bundles

### Packages with many exports (icon/utility libraries)
```js
// next.config.js
module.exports = { experimental: { optimizePackageImports: ['icon-library'] } }
```
Only loads modules actually used. Many libraries auto-optimized already (check the full list in docs).

### Heavy client workloads — move to Server Components
```tsx
// BAD: ships prism to client
'use client'
import Highlight from 'prism-react-renderer'

// GOOD: runs on server, ships only HTML
import { codeToHtml } from 'shiki'
export default async function Page() {
  const html = await codeToHtml(code, { lang: 'tsx', theme: 'github-dark' })
  return <pre><code dangerouslySetInnerHTML={{ __html: html }} /></pre>
}
```

### Opt packages out of Server Component bundling
```js
// next.config.js
module.exports = { serverExternalPackages: ['package-name'] }
```

## Tools for bundle analysis
- [Import Cost](https://marketplace.visualstudio.com/items?itemName=wix.vscode-import-cost) — VS Code extension
- [Package Phobia](https://packagephobia.com/) — install size
- [Bundle Phobia](https://bundlephobia.com/) — bundle size
- [bundlejs](https://bundlejs.com/) — online bundle analyzer
- [Dependency Cruiser](https://github.com/sverweij/dependency-cruiser) — import graph visualization
- [Madge](https://github.com/pahen/madge) — circular dependency detection