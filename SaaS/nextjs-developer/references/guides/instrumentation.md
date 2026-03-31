# Instrumentation

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/instrumentation

Run code once at server startup (monitoring, logging, OpenTelemetry setup) before the server handles requests.

## Convention

Create `instrumentation.ts|js` in the **project root** (or `src/` if using src folder). Export a `register` function:

```ts
// instrumentation.ts
import { registerOTel } from '@vercel/otel'

export function register() {
  registerOTel('next-app')
}
```

- NOT inside `app/` or `pages/` directories
- If using `pageExtensions` config, update the filename to match
- Called once per new server instance, must complete before server is ready

## Importing files with side effects

Import inside `register()` (not at top level) to colocate side effects:
```ts
export async function register() {
  await import('package-with-side-effect')
}
```

## Runtime-specific code

Use `NEXT_RUNTIME` to conditionally import:
```ts
export async function register() {
  if (process.env.NEXT_RUNTIME === 'nodejs') {
    await import('./instrumentation-node')
  }
  if (process.env.NEXT_RUNTIME === 'edge') {
    await import('./instrumentation-edge')
  }
}
```

See [OpenTelemetry guide](/docs/app/guides/open-telemetry) for full setup.