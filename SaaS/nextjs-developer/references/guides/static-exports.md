# Static Exports

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/static-exports

## Configuration

```js
// next.config.js
module.exports = {
  output: 'export',
  // Optional:
  trailingSlash: true,      // /me → /me/ + /me/index.html
  distDir: 'dist',          // change output dir from 'out'
}
```

`next build` generates an `out/` folder with HTML/CSS/JS.

## Supported Features

- **Server Components** — run at build time, render to static HTML
- **Client Components** — prerendered, then hydrated client-side
- **Route Handlers** — `GET` only with `dynamic = 'force-static'`
- **Image Optimization** — with custom loader (not the default)
- **Static image imports**, **CSS Modules**, **Global CSS**, **`next/link`**, **`next/script`**

### Custom image loader example (Cloudinary)
```js
module.exports = { output: 'export', images: { loader: 'custom', loaderFile: './my-loader.ts' } }
```

## Unsupported Features (require a server)

Dynamic Routes with `dynamicParams: true`, Route Handlers using `Request`, `cookies()`, `headers()`, Rewrites, Redirects, Headers, Proxy/Middleware, ISR, Draft Mode, Server Actions, Intercepting Routes, default image loader.

## Deploying

Generated `out/` can be hosted on any static server (AWS S3, Nginx, Apache, GitHub Pages).

### Nginx config example
```nginx
location / {
  try_files $uri $uri.html $uri/ =404;
}
location /blog/ {
  rewrite ^/blog/(.*)$ /blog/$1.html break;  # needed when trailingSlash: false
}
error_page 404 /404.html;
```

## Version History

| Version | Change |
|---|---|
| v14.0.0 | `next export` removed; use `output: 'export'` |
| v13.4.0 | App Router + Server Components + Route Handlers support added |
| v13.3.0 | `next export` deprecated |