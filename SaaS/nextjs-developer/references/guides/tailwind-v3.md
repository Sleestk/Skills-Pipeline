# Tailwind CSS v3

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/tailwind-v3-css

Use Tailwind v3 for broader browser support. For the latest Tailwind v4 setup, see the [CSS guide](/docs/app/getting-started/css#tailwind-css).

## Install

```bash
pnpm add -D tailwindcss@^3 postcss autoprefixer
npx tailwindcss init -p  # generates tailwind.config.js + postcss.config.js
```

## Configure `tailwind.config.js`

```js
module.exports = {
  content: [
    './app/**/*.{js,ts,jsx,tsx,mdx}',
    './pages/**/*.{js,ts,jsx,tsx,mdx}',
    './components/**/*.{js,ts,jsx,tsx,mdx}',
  ],
  theme: { extend: {} },
  plugins: [],
}
```

## Global CSS

```css
/* app/globals.css */
@tailwind base;
@tailwind components;
@tailwind utilities;
```

## Root Layout

```tsx
// app/layout.tsx
import './globals.css'
export default function RootLayout({ children }) {
  return <html lang="en"><body>{children}</body></html>
}
```

## Usage

```tsx
export default function Page() {
  return <h1 className="text-3xl font-bold underline">Hello, Next.js!</h1>
}
```

Turbopack supports Tailwind CSS and PostCSS as of Next.js 13.1.