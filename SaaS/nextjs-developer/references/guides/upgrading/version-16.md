# Upgrading to v16

> https://nextjs.org/docs/app/guides/upgrading/version-16

```bash
pnpm next upgrade        # requires v16.1.0+ — upgrades to latest
# or:
npx @next/codemod@canary upgrade latest
```

## Key Changes in v16

### `middleware.ts` → `proxy.ts` (Breaking)
Proxy replaces Middleware. The file convention, export name, and config options all change.

**Codemod**: `middleware-to-proxy`

```ts
// Before: middleware.ts
export function middleware(request) { return NextResponse.next() }

// After: proxy.ts
export function proxy(request) { return NextResponse.next() }
```

Config renames in `next.config.js`:
- `experimental.middlewarePrefetch` → `experimental.proxyPrefetch`
- `experimental.middlewareClientMaxBodySize` → `experimental.proxyClientMaxBodySize`
- `experimental.externalMiddlewareRewritesResolve` → `experimental.externalProxyRewritesResolve`
- `skipMiddlewareUrlNormalize` → `skipProxyUrlNormalize`

### `next lint` removed
Run linter via ESLint CLI directly. **Codemod**: `next-lint-to-eslint-cli`

```json
// Before
{ "scripts": { "lint": "next lint" } }
// After
{ "scripts": { "lint": "eslint ." } }
```

`next build` no longer runs the linter automatically.

### Turbopack default in dev
`next dev` uses Turbopack by default. Use `next dev --webpack` to opt back to Webpack.

### `experimental_ppr` removed
Use `ppr: true` in `next.config.js` instead. **Codemod**: `remove-experimental-ppr`

### Stabilized APIs
`unstable_` prefix removed from `cacheTag`, `cacheLife`, and others. **Codemod**: `remove-unstable-prefix`

### Cache Components (`use cache`)
Available via `cacheComponents: true` in `next.config.js`. Replaces route segment configs (`dynamic`, `revalidate`, `fetchCache`) with `'use cache'` directive + `cacheLife()`. See [Migrating to Cache Components](/docs/app/guides/migrating-to-cache-components).

### Next.js MCP Server
Built-in MCP endpoint at `/_next/mcp`. Use `next-devtools-mcp` package with coding agents. See [MCP Server guide](/docs/app/guides/mcp).

### `AGENTS.md` / `CLAUDE.md`
`create-next-app` now generates agent instruction files automatically. See [AI Coding Agents guide](/docs/app/guides/ai-agents).

### Node.js middleware runtime (experimental)
Proxy can now run on full Node.js runtime (previously Edge-only):
```ts
export const config = { runtime: 'nodejs' }
```