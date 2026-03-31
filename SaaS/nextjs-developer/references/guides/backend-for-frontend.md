# Backend for Frontend

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/backend-for-frontend

Next.js supports the BFF pattern via Route Handlers (`route.ts`) and `proxy.ts`. Start with `--api` flag: `npx create-next-app@latest --api`.

## Public Endpoints (Route Handlers)

```ts
// app/api/route.ts
export function GET(request: Request) {}
export async function POST(request: Request) {
  try {
    await submit(request)
    return new Response(null, { status: 204 })
  } catch (reason) {
    const message = reason instanceof Error ? reason.message : 'Unexpected error'
    return new Response(message, { status: 500 })
  }
}
```

- Avoid exposing sensitive info in error messages
- Implement auth/authz — see [Authentication](/docs/app/guides/authentication)

## Content Types

Serve JSON, XML, images, files, plain text. File conventions (`sitemap.xml`, `opengraph-image`, `manifest.json`, `robots.txt`) are built-in. Custom: `llms.txt`, `rss.xml`, `.well-known`.

### Content Negotiation (serve different types from same URL)
Use rewrites with header matching in `next.config.js` to serve Markdown to AI agents and HTML to browsers from the same `/docs/...` URL. Add `Vary: Accept` response header so CDNs cache them separately.

## Consuming Request Payloads
```ts
const res = await request.json()      // JSON
const formData = await request.formData() // FormData
const text = await request.text()     // Text
// Can only read body once — clone first: const cloned = request.clone()
```

## Proxying to a Backend
```ts
export async function POST(request: Request, { params }) {
  const cloned = request.clone()
  if (!await isValidRequest(cloned)) return new Response(null, { status: 400 })
  const proxyURL = new URL(slug.join('/'), 'https://target.com')
  return fetch(new Request(proxyURL, request))
}
```

## NextRequest / NextResponse
- `NextRequest.nextUrl` — parsed URL with pathname, searchParams, etc.
- `NextResponse.json()`, `redirect()`, `rewrite()`, `next()`

## Webhooks & Callbacks
Receive CMS events, revalidate cache on change, set session cookies from OAuth callbacks.

## Proxy (`proxy.ts`)
```ts
// proxy.ts — runs before every route
import { isAuthenticated } from '@lib/auth'
export const config = { matcher: '/api/:function*' }
export function proxy(request: Request) {
  if (!isAuthenticated(request)) {
    return Response.json({ success: false, message: 'authentication failed' }, { status: 401 })
  }
}
```

## Security
- **Headers**: upstream request headers (`NextResponse.next({ request: { headers } })`) stay server-side; response headers go to client — don't leak sensitive values
- **Rate limiting**: implement in Route Handlers + use host-provided rate limiting
- **Validate payloads**: content type, size, sanitize XSS
- **Auth on every request**: don't rely on proxy alone

## Caveats
- **Server Components**: fetch directly from data source, not via Route Handlers (extra HTTP round trip)
- **Static export**: only `GET` with `dynamic = 'force-static'` supported
- **Serverless**: no shared state between requests, no filesystem writes, WebSockets won't persist