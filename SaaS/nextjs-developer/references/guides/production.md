# Production Checklist

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/production-checklist

## Automatic Optimizations (no config needed)
- **Server Components** by default — no client JS for rendering
- **Code-splitting** by route segment
- **Prefetching** via `<Link>` in viewport
- **Prerendering** at build time
- **Caching** of data, rendered output, and static assets

## Routing and Rendering
- Use `layout.tsx` for shared UI and partial rendering on nav
- Use `<Link>` for client-side nav + prefetching
- Custom error pages: `error.tsx`, `not-found.tsx`, `global-error.tsx`, `global-not-found.tsx`
- Check `'use client'` boundaries — push them down to reduce bundle size
- Be intentional about Request-time APIs (`cookies`, `searchParams`) — they opt entire routes into dynamic rendering; wrap in `<Suspense>`

## Data Fetching and Caching
- Fetch in Server Components, not via Route Handlers from Server Components
- Use `loading.tsx` / `<Suspense>` for streaming
- Use `Promise.all` for parallel fetches
- Verify what is/isn't cached; use `unstable_cache` for non-`fetch` functions
- Cache static assets in `public/`

## UI and Accessibility
- Server Actions for form submissions + server-side validation
- `app/global-error.tsx` — consistent fallback for uncaught errors
- `app/global-not-found.tsx` — accessible 404 for unmatched routes
- `<Font>` component — self-hosted fonts, no external requests, prevents layout shift
- `<Image>` component — auto-optimization, prevents layout shift, WebP format
- `<Script>` component — deferred loading, no main thread blocking
- `eslint-plugin-jsx-a11y` — catch accessibility issues early

## Security
- **Tainting** — `experimental.taint: true` to prevent sensitive data reaching client
- **Server Actions** — auth + authz inside every action; DAL for data access; rate-limit expensive ops
- **Env vars** — `.env.*` in `.gitignore`; only `NEXT_PUBLIC_` exposed to browser
- **CSP** — consider Content Security Policy for XSS/clickjacking protection

## Metadata and SEO
- **Metadata API** — `generateMetadata()` or static `metadata` export for titles, descriptions
- **OG images** — `opengraph-image` file convention or `ImageResponse`
- **Sitemaps** — `sitemap.ts` or `generateSitemaps()`
- **Robots** — `robots.ts`

## Type Safety
- TypeScript + TS Plugin for auto-completion and type-checking
- Enable VS Code workspace TypeScript version for full Next.js plugin benefits

## Before Deploying
- Run `next build` locally to catch build errors
- Run `next start` to test production behavior
- Run Lighthouse in incognito for Core Web Vitals baseline
- Use `@next/bundle-analyzer` to check bundle sizes