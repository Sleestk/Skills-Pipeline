# Development Environment

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/local-development

`next dev` compiles routes on demand (only when visited), unlike `next build` which compiles everything.

## Performance Fixes

### 1. Antivirus
- **Windows**: add project folder to Microsoft Defender exclusion list (Windows Security → Virus & threat protection → Manage settings → Add or remove exclusions)
- **macOS**: `sudo spctl developer-mode enable-terminal` → System Settings → Privacy & Security → Developer Tools → enable terminal

### 2. Update Next.js + use Turbopack
```bash
pnpm add next@latest
pnpm dev  # Turbopack is the default
```
Use Webpack if needed: `next dev --webpack`

### 3. Check imports

**Icon libraries** — import specific icons, not the entire library:
```jsx
// Bad
import { TriangleIcon } from '@phosphor-icons/react'
// Good
import { TriangleIcon } from '@phosphor-icons/react/dist/csr/Triangle'
```

**Barrel files** — import directly from specific files when possible. Use `optimizePackageImports` for packages with barrel files:
```js
// next.config.js
module.exports = { experimental: { optimizePackageImports: ['package-name'] } }
```

Turbopack auto-analyzes and optimizes imports — no config needed.

### 4. Tailwind CSS content paths
Be specific — avoid scanning `node_modules`:
```js
// tailwind.config.js
module.exports = {
  content: ['./src/**/*.{js,ts,jsx,tsx}'],  // Good — specific
  // '../../packages/**/*.{js,ts,jsx,tsx}'  // Bad — may match node_modules
}
```

### 5. Custom webpack settings
Remove unnecessary webpack plugins from dev. Consider switching to Turbopack defaults.

### 6. Memory usage
See [Memory Usage](/docs/app/guides/memory-usage) guide.

### 7. Server Components HMR cache
Cache `fetch` responses across HMR refreshes (experimental):
```js
// next.config.js
module.exports = { experimental: { serverComponentsHmrCache: true } }
```

### 8. Avoid Docker on Mac/Windows for dev
Docker filesystem on Mac/Windows causes very slow HMR. Use local dev; reserve Docker for production.

## Tools

### Fetch logging
```js
module.exports = { logging: { fetches: { fullUrl: true } } }
```

### Turbopack tracing
```bash
NEXT_TURBOPACK_TRACING=1 pnpm dev
# Navigate/edit to reproduce issue, then stop server
npx next internal trace .next/dev/trace-turbopack
# View at https://trace.nextjs.org/
```

Switch from "Aggregated in order" to "Spans in order" in the viewer for individual timings.