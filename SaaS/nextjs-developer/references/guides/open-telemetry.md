# OpenTelemetry

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/open-telemetry

## Quick Setup with `@vercel/otel`

```bash
pnpm add @vercel/otel @opentelemetry/sdk-logs @opentelemetry/api-logs @opentelemetry/instrumentation
```

```ts
// instrumentation.ts
import { registerOTel } from '@vercel/otel'
export function register() {
  registerOTel({ serviceName: 'next-app' })
}
```

## Manual Setup (Node.js only)

```ts
// instrumentation.ts
export async function register() {
  if (process.env.NEXT_RUNTIME === 'nodejs') {
    await import('./instrumentation.node.ts')
  }
}

// instrumentation.node.ts
import { OTLPTraceExporter } from '@opentelemetry/exporter-trace-otlp-http'
import { resourceFromAttributes } from '@opentelemetry/resources'
import { NodeSDK } from '@opentelemetry/sdk-node'
import { SimpleSpanProcessor } from '@opentelemetry/sdk-trace-node'
import { ATTR_SERVICE_NAME } from '@opentelemetry/semantic-conventions'

const sdk = new NodeSDK({
  resource: resourceFromAttributes({ [ATTR_SERVICE_NAME]: 'next-app' }),
  spanProcessor: new SimpleSpanProcessor(new OTLPTraceExporter()),
})
sdk.start()
```

## Custom Spans

```ts
import { trace } from '@opentelemetry/api'

export async function fetchGithubStars() {
  return await trace.getTracer('nextjs-example')
    .startActiveSpan('fetchGithubStars', async (span) => {
      try { return await getValue() }
      finally { span.end() }
    })
}
```

## Built-in Next.js Spans

| Span | Type | Attributes |
|---|---|---|
| `[http.method] [next.route]` | `BaseServer.handleRequest` | `http.method`, `http.status_code`, `http.route` |
| `render route (app) [route]` | `AppRender.getBodyResult` | `next.route` |
| `fetch [method] [url]` | `AppRender.fetch` | `http.method`, `http.url` |
| `executing api route (app) [route]` | `AppRouteRouteHandlers.runHandler` | `next.route` |
| `getServerSideProps [route]` | `Render.getServerSideProps` | `next.route` |
| `getStaticProps [route]` | `Render.getStaticProps` | `next.route` |
| `generateMetadata [page]` | `ResolveMetadata.generateMetadata` | `next.page` |
| `resolve page components` | `NextNodeServer.findPageComponents` | `next.route` |
| `start response` | `NextNodeServer.startResponse` | — |

Disable fetch spans: `NEXT_OTEL_FETCH_DISABLED=1`. Enable verbose: `NEXT_OTEL_VERBOSE=1`.

## Testing Locally
Use [OpenTelemetry dev environment](https://github.com/vercel/opentelemetry-collector-dev-setup). Check for root server span labeled `GET /requested/pathname`.

## Deployment
- **Vercel**: works out of the box, follow [Vercel observability quickstart](https://vercel.com/docs/concepts/observability/otel-overview/quickstart)
- **Self-hosted**: run OpenTelemetry Collector, configure to receive from Next.js app