# CSS

> Next.js 16.2.1 — https://nextjs.org/docs/app/getting-started/css

Next.js supports: Tailwind CSS, CSS Modules, Global CSS, External Stylesheets, Sass, CSS-in-JS.

---

## Tailwind CSS

Install:
```bash
pnpm add -D tailwindcss @tailwindcss/postcss
```

Configure PostCSS:
```js
// postcss.config.mjs
export default {
  plugins: { '@tailwindcss/postcss': {} },
}
```

Import in global CSS:
```css
/* app/globals.css */
@import 'tailwindcss';
```

Import in root layout:
```tsx
// app/layout.tsx
import './globals.css'
export default function RootLayout({ children }: { children: React.ReactNode }) {
  return <html lang="en"><body>{children}</body></html>
}
```

> For broader browser support with older browsers, see the [Tailwind CSS v3 guide](https://nextjs.org/docs/app/guides/tailwind-v3-css).

---

## CSS Modules

Files ending in `.module.css` are locally scoped — unique class names are generated at build time, preventing collisions:

```css
/* app/blog/blog.module.css */
.blog { padding: 24px; }
```

```tsx
// app/blog/page.tsx
import styles from './blog.module.css'
export default function Page() {
  return <main className={styles.blog}></main>
}
```

---

## Global CSS

Import a global stylesheet in the root layout to apply styles to every route:

```css
/* app/global.css */
body {
  padding: 20px 20px 60px;
  max-width: 680px;
  margin: 0 auto;
}
```

```tsx
// app/layout.tsx
import './global.css'
export default function RootLayout({ children }: { children: React.ReactNode }) {
  return <html lang="en"><body>{children}</body></html>
}
```

> Global styles can be imported into any layout, page, or component in `app/`. However, Next.js currently does **not** remove stylesheets on navigation (React's Suspense integration), which can cause conflicts. Use global styles only for truly global CSS (like Tailwind base styles).

---

## External Stylesheets

Import directly from `node_modules` anywhere in `app/`:

```tsx
// app/layout.tsx
import 'bootstrap/dist/css/bootstrap.css'
```

> In React 19, you can also use `<link rel="stylesheet" href="..." />` directly.

---

## Ordering and Merging

CSS order in production = **import order in your code**.

```tsx
// page.tsx
import { BaseButton } from './base-button'  // base-button.module.css loads first
import styles from './page.module.css'       // page.module.css loads second
```

### Recommendations
- Contain CSS imports to a single entry file where possible.
- Import global styles and Tailwind in the root layout.
- Use **Tailwind CSS** for most styling needs.
- Use **CSS Modules** for custom scoped styles when Tailwind isn't sufficient.
- Use consistent naming: `<name>.module.css`.
- Extract shared styles into shared components.
- Disable auto-sort-import linters (e.g. ESLint's `sort-imports`) — they break CSS order.
- Use [`cssChunking`](https://nextjs.org/docs/app/api-reference/config/next-config-js/cssChunking) in `next.config.js` to control chunking behavior.

---

## Dev vs Production

| | Development | Production |
|---|---|---|
| CSS updates | Instant via Fast Refresh | Minified + code-split `.css` files |
| JS required | Yes (for Fast Refresh) | No (CSS loads without JS) |
| CSS ordering | May differ from production | Final order — always verify with `next build` |