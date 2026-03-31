# Sass

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/sass

Built-in support for `.scss`/`.sass` (global) and `.module.scss`/`.module.sass` (CSS Modules).

## Install
```bash
pnpm add -D sass
```

Use `.scss` (superset of CSS, recommended for beginners) or `.sass` (indented syntax).

## Configure Sass options
```ts
// next.config.ts
const nextConfig: NextConfig = {
  sassOptions: {
    additionalData: `$var: red;`,
    implementation: 'sass-embedded',  // optional: use sass-embedded for better performance
  },
}
```

## Sass Variables in CSS Modules

Export Sass variables for use in JS/TS:
```scss
/* app/variables.module.scss */
$primary-color: #64ff00;
:export { primaryColor: $primary-color; }
```

```jsx
// app/page.js
import variables from './variables.module.scss'
export default function Page() {
  return <h1 style={{ color: variables.primaryColor }}>Hello!</h1>
}
```