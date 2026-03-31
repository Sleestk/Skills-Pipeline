# Content Security Policy (CSP)

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/content-security-policy

## With Nonces (Dynamic Rendering Required)

Nonces require **dynamic rendering** — pages can't be cached/statically generated when using nonces. PPR is also incompatible with nonce-based CSP.

### Proxy generates nonce per request
```ts
// proxy.ts
import { NextRequest, NextResponse } from 'next/server'
export function proxy(request: NextRequest) {
  const nonce = Buffer.from(crypto.randomUUID()).toString('base64')
  const isDev = process.env.NODE_ENV === 'development'
  const cspHeader = `
    default-src 'self';
    script-src 'self' 'nonce-${nonce}' 'strict-dynamic'${isDev ? " 'unsafe-eval'" : ''};
    style-src 'self' 'nonce-${nonce}';
    img-src 'self' blob: data:;
    font-src 'self';
    object-src 'none';
    base-uri 'self';
    form-action 'self';
    frame-ancestors 'none';
    upgrade-insecure-requests;
  `.replace(/\s{2,}/g, ' ').trim()

  const requestHeaders = new Headers(request.headers)
  requestHeaders.set('x-nonce', nonce)
  requestHeaders.set('Content-Security-Policy', cspHeader)

  const response = NextResponse.next({ request: { headers: requestHeaders } })
  response.headers.set('Content-Security-Policy', cspHeader)
  return response
}

export const config = {
  matcher: [{
    source: '/((?!api|_next/static|_next/image|favicon.ico).*)',
    missing: [
      { type: 'header', key: 'next-router-prefetch' },
      { type: 'header', key: 'purpose', value: 'prefetch' },
    ],
  }],
}
```

### Force dynamic rendering for nonces
```tsx
import { connection } from 'next/server'
export default async function Page() {
  await connection()
  // page content
}
```

### Read nonce in Server Component
```tsx
import { headers } from 'next/headers'
import Script from 'next/script'
export default async function Page() {
  const nonce = (await headers()).get('x-nonce')
  return <Script src="https://www.googletagmanager.com/gtag/js" strategy="afterInteractive" nonce={nonce} />
}
```

Next.js automatically attaches the nonce to framework scripts, page bundles, and inline styles/scripts.

> In development, `'unsafe-eval'` is required (React uses eval for debugging). Not needed in production.

## Without Nonces (Static-Compatible)

```js
// next.config.js
const isDev = process.env.NODE_ENV === 'development'
const cspHeader = `
  default-src 'self';
  script-src 'self' 'unsafe-inline'${isDev ? " 'unsafe-eval'" : ''};
  style-src 'self' 'unsafe-inline';
  img-src 'self' blob: data:;
  font-src 'self'; object-src 'none';
  base-uri 'self'; form-action 'self';
  frame-ancestors 'none'; upgrade-insecure-requests;
`
module.exports = {
  async headers() {
    return [{ source: '/(.*)', headers: [{ key: 'Content-Security-Policy', value: cspHeader.replace(/\n/g, '') }] }]
  },
}
```

## Subresource Integrity (Experimental — Static-Compatible Alternative)

```js
// next.config.js
module.exports = {
  experimental: { sri: { algorithm: 'sha256' } }, // or 'sha384', 'sha512'
}
```

SRI generates build-time hashes and adds `integrity` attributes to script tags. Works with static generation and CDN caching.

## Third-Party Scripts with Nonces
```tsx
import { GoogleTagManager } from '@next/third-parties/google'
const nonce = (await headers()).get('x-nonce')
// Update CSP to allow GTM domain:
// script-src ... https://www.googletagmanager.com
<GoogleTagManager gtmId="GTM-XYZ" nonce={nonce} />
```

## Common CSP Violations
- **Inline styles** — use CSS-in-JS with nonce support or external files
- **Dynamic imports** — ensure script-src policy allows them
- **WebAssembly** — add `'wasm-unsafe-eval'`
- **Service workers** — add appropriate script policies