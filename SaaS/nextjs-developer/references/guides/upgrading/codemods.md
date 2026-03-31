# Codemods

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/upgrading/codemods

```bash
npx @next/codemod <transform> <path> [--dry] [--print]
```

## `upgrade` command (auto-run codemods + update deps)
```bash
npx @next/codemod upgrade [patch|minor|major|canary|<version>]
# Examples:
npx @next/codemod upgrade minor   # default
npx @next/codemod upgrade 16
npx @next/codemod upgrade canary
```

---

## v16 Codemods

### `remove-experimental-ppr`
Removes `experimental_ppr` route segment config from App Router pages/layouts.

### `remove-unstable-prefix`
Removes `unstable_` prefix from stabilized APIs (e.g. `unstable_cacheTag` → `cacheTag`).

### `middleware-to-proxy`
Renames `middleware.ts` → `proxy.ts`, `export function middleware` → `export function proxy`, and updates all related `next.config.js` property names (`middlewarePrefetch` → `proxyPrefetch`, etc.).

### `next-lint-to-eslint-cli`
Migrates from `next lint` to ESLint CLI. Updates `package.json` scripts, creates `eslint.config.mjs`, adds ESLint deps.

---

## v15 Codemods

### `next-async-request-api`
Transforms `cookies()`, `headers()`, `draftMode()`, `params`, `searchParams` to be properly awaited or wrapped with `React.use()`.

### `app-dir-runtime-config-experimental-edge`
Changes `runtime = 'experimental-edge'` → `runtime = 'edge'`.

### `next-request-geo-ip`
Replaces `req.geo` and `req.ip` with `@vercel/functions` equivalents.

---

## v14 Codemods

### `next-og-import`
Moves `ImageResponse` import from `next/server` → `next/og`.

### `metadata-to-viewport-export`
Migrates `themeColor`/`viewport` from `metadata` export to separate `viewport` export.

---

## v13 Codemods

### `built-in-next-font`
Uninstalls `@next/font`, changes imports to `next/font`.

### `next-image-to-legacy-image`
Renames `next/image` → `next/legacy/image` and `next/future/image` → `next/image`.

### `new-link`
Removes `<a>` tags inside `<Link>` components.

---

## Older Codemods

### `cra-to-next` (v11)
Migrates Create React App project to Next.js Pages Router.

### `name-default-component` (v9)
Converts anonymous default exports to named components (required for Fast Refresh).

### `url-to-withrouter` (v6)
Replaces deprecated `this.props.url` with `withRouter` HOC.