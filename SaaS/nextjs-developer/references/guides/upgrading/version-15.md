# Upgrading to v15

> https://nextjs.org/docs/app/guides/upgrading/version-15

```bash
npx @next/codemod@canary upgrade latest
# or manually:
npm i next@latest react@latest react-dom@latest eslint-config-next@latest
```

## React 19

- Minimum `react` and `react-dom` version is now **19**
- `useFormState` deprecated → use `useActionState` (includes `pending` state directly)
- `useFormStatus` now exposes `data`, `method`, `action` (React 19 only)
- Update `@types/react` and `@types/react-dom` to latest

## Async Request APIs (Breaking)

These previously synchronous APIs are now **async** — must be awaited:

- `cookies()`, `headers()`, `draftMode()` from `next/headers`
- `params` in `layout.js`, `page.js`, `route.js`, `default.js`, metadata files
- `searchParams` in `page.js`

**Codemod**: `next-async-request-api` handles most cases automatically.

```ts
// Before
const cookieStore = cookies()
const token = cookieStore.get('token')

// After
const cookieStore = await cookies()
const token = cookieStore.get('token')
```

For Client Components use `React.use()`:
```tsx
import { use } from 'react'
export default function Layout(props) {
  const params = use(props.params)
}
```

Temporary synchronous access (logs warning in dev): cast to `UnsafeUnwrappedCookies`.

## Other Breaking Changes

- **`runtime = 'experimental-edge'`** → use `'edge'` — codemod: `app-dir-runtime-config-experimental-edge`
- **`fetch` no longer cached by default** — opt in with `cache: 'force-cache'`
- **Route Handler `GET` no longer cached** — opt in with `export const dynamic = 'force-static'`
- **Client Cache** — page segments no longer reused from cache on navigation (only layouts/loading still cached). Use `staleTimes` config to opt back in.
- **`@next/font` removed** → use `next/font` — codemod: `built-in-next-font`
- **`bundlePagesExternals`** → `bundlePagesRouterDependencies`
- **`serverComponentsExternalPackages`** → `serverExternalPackages`
- **Speed Insights auto-instrumentation removed** — use Vercel Speed Insights manually
- **`NextRequest` `geo`/`ip` properties removed** — use `@vercel/functions` — codemod: `next-request-geo-ip`