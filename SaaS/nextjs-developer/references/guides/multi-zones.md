# Multi-Zones

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/multi-zones

Micro-frontend pattern: separate Next.js apps deployed under one domain, each serving different path prefixes.

Example split: `/blog/*` + `/dashboard/*` + `/*` = 3 independent apps.

- Same-zone navigation = soft (client-side, no reload)
- Cross-zone navigation = hard (full page reload) — keep frequently co-visited pages in same zone

## Define a zone

```js
// next.config.js (blog zone)
module.exports = { assetPrefix: '/blog-static' }
```

The `assetPrefix` prevents asset conflicts. Default zone (catch-all) needs no prefix.

## Route requests to the right zone

In the main/default zone, add `rewrites`:
```js
// next.config.js (main zone)
async rewrites() {
  return [
    { source: '/blog', destination: `${process.env.BLOG_DOMAIN}/blog` },
    { source: '/blog/:path+', destination: `${process.env.BLOG_DOMAIN}/blog/:path+` },
    { source: '/blog-static/:path+', destination: `${process.env.BLOG_DOMAIN}/blog-static/:path+` },
  ]
}
```

Or use `proxy.ts` for dynamic routing decisions (feature flags, A/B testing).

## Linking between zones

Use `<a>` tags (not `<Link>`) for cross-zone navigation — `<Link>` will try to prefetch/soft-navigate which doesn't work across zones.

## Sharing code

Use a monorepo or publish shared code as npm packages.

## Server Actions with Multi-Zones

Add `allowedOrigins` to prevent CSRF:
```js
module.exports = {
  experimental: { serverActions: { allowedOrigins: ['your-production-domain.com'] } }
}
```

Example: https://github.com/vercel/next.js/tree/canary/examples/with-zones