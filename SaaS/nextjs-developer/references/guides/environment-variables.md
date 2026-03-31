# Environment Variables

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/environment-variables

## Loading from `.env` files

```txt
# .env
DB_HOST=localhost
DB_USER=myuser
DB_PASS=mypassword
```

Variables auto-loaded into `process.env`. Supports multiline with `\n` or literal line breaks.

> If using `/src` folder, `.env` files must stay in the project root, not in `/src`.

## Load order (stops at first match)
1. `process.env`
2. `.env.$(NODE_ENV).local`
3. `.env.local` *(not checked when `NODE_ENV=test`)*
4. `.env.$(NODE_ENV)`
5. `.env`

## Browser exposure — `NEXT_PUBLIC_` prefix

Vars without `NEXT_PUBLIC_` are server-only. Prefix with `NEXT_PUBLIC_` to inline into client JS bundle at build time:

```txt
NEXT_PUBLIC_ANALYTICS_ID=abcdefghijk
```

> After building, `NEXT_PUBLIC_` vars are frozen. Changes require a rebuild.

Dynamic lookups are NOT inlined:
```js
// NOT inlined:
const varName = 'NEXT_PUBLIC_ANALYTICS_ID'
setupAnalyticsService(process.env[varName])
```

## Runtime environment variables

Read server-side env vars safely during dynamic rendering:
```tsx
import { connection } from 'next/server'
export default async function Component() {
  await connection() // opts into dynamic rendering
  const value = process.env.MY_VALUE // evaluated at runtime
}
```

## Loading outside Next.js runtime (ORMs, test runners)

```ts
import { loadEnvConfig } from '@next/env'
loadEnvConfig(process.cwd())
```

Install `@next/env` for this utility.

## Variable expansion

```txt
TWITTER_USER=nextjs
TWITTER_URL=https://x.com/$TWITTER_USER  # → https://x.com/nextjs
```

Escape literal `$` with `\$`.

## Test environment

`.env.test` for test-specific values (`NODE_ENV=test`). `.env.local` is NOT loaded in test environments.

For Jest global setup:
```js
import { loadEnvConfig } from '@next/env'
export default async () => { loadEnvConfig(process.cwd()) }
```

## NODE_ENV
- `next dev` → `development`
- All other commands → `production`
- Valid values: `production`, `development`, `test`