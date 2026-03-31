# Next.js Components Reference (v16.2.1)

Built-in components from `next/image`, `next/link`, `next/font`, `next/form`, `next/script`.

---

## `next/image` — Image Optimization

```tsx
import Image from 'next/image'

<Image
  src="/profile.png"
  width={500}
  height={500}
  alt="Description"
/>
```

### Required Props

| Prop | Notes |
|---|---|
| `src` | Internal path, absolute external URL (requires `remotePatterns`), or static import |
| `alt` | Required. Empty string `alt=""` for decorative images |
| `width` + `height` | Required unless using `fill` or static import |

### Key Props

#### `fill`
Expands image to fill parent. Parent **must** have `position: relative/fixed/absolute`.
```tsx
<div style={{ position: 'relative', width: '400px', height: '300px' }}>
  <Image fill src="/hero.jpg" alt="Hero" style={{ objectFit: 'cover' }} />
</div>
```

#### `sizes`
Tell the browser how wide the image is at each breakpoint. Required with `fill` or responsive images.
```tsx
<Image
  fill
  src="/photo.jpg"
  sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
/>
```
- **Without `sizes`**: generates limited `srcset` (1x, 2x) — for fixed-size images
- **With `sizes`**: generates full `srcset` (640w, 750w, …) — for responsive layouts

#### `preload` (v16+)
Injects `<link>` in `<head>` to preload the image. Use for LCP/hero images.
```tsx
<Image src="/hero.jpg" width={1200} height={600} alt="Hero" preload />
```
> **v16**: `priority` is **deprecated** — use `preload` instead.

#### `loading`
```tsx
<Image loading="lazy" />   // default — defers until near viewport
<Image loading="eager" />  // loads immediately
```

#### `placeholder` + `blurDataURL`
```tsx
// Static import: blurDataURL added automatically
import mountains from './mountains.jpg'
<Image src={mountains} placeholder="blur" alt="Mountains" />

// Remote/dynamic: must provide blurDataURL manually
<Image
  src="https://example.com/photo.jpg"
  placeholder="blur"
  blurDataURL="data:image/jpeg;base64,/9j/4AA..."
  width={800}
  height={600}
  alt="Photo"
/>
```

#### `quality`
Integer 1–100. Default `75`. Must match a value in `next.config.js` `qualities` array (required in v16).
```tsx
<Image quality={90} src="..." width={800} height={600} alt="..." />
```

#### `unoptimized`
Serves image as-is. Useful for SVGs, GIFs, tiny images.
```tsx
<Image src="/icon.svg" unoptimized width={24} height={24} alt="Icon" />
```

#### `loader`
Custom URL generation function. Must be in a Client Component.
```tsx
'use client'
const myLoader = ({ src, width, quality }) =>
  `https://cdn.example.com/${src}?w=${width}&q=${quality || 75}`

<Image loader={myLoader} src="profile.jpg" width={500} height={500} alt="..." />
```

#### `decoding`
```tsx
<Image decoding="async" />  // default
<Image decoding="sync" />   // atomic with other content
```

#### `overrideSrc`
Overrides the generated `src` attribute while keeping the optimized `srcset`. Use when migrating from `<img>` for SEO continuity.
```tsx
<Image src="/profile.jpg" overrideSrc="/legacy/profile.jpg" ... />
```

#### `onLoad` / `onError`
Require Client Components.
```tsx
'use client'
<Image onLoad={(e) => console.log(e.target.naturalWidth)} src="..." ... />
<Image onError={(e) => console.error('Failed', e)} src="..." ... />
```

### `getImageProps()`
Get the raw `<img>` props without rendering a component. Useful for art direction, background CSS, or custom elements.
```tsx
import { getImageProps } from 'next/image'

// Art direction (mobile vs desktop)
const { props: { srcSet: desktop } } = getImageProps({ src: '/desktop.jpg', width: 1440, height: 875, quality: 80, alt: '' })
const { props: { srcSet: mobile, ...rest } } = getImageProps({ src: '/mobile.jpg', width: 750, height: 1334, quality: 70, alt: '' })

return (
  <picture>
    <source media="(min-width: 1000px)" srcSet={desktop} />
    <source media="(min-width: 500px)" srcSet={mobile} />
    <img {...rest} style={{ width: '100%', height: 'auto' }} />
  </picture>
)

// Background CSS
function getBackgroundImage(srcSet = '') {
  const imageSet = srcSet.split(', ').map((str) => {
    const [url, dpi] = str.split(' ')
    return `url("${url}") ${dpi}`
  }).join(', ')
  return `image-set(${imageSet})`
}
const { props: { srcSet } } = getImageProps({ alt: '', width: 128, height: 128, src: '/img.png' })
const backgroundImage = getBackgroundImage(srcSet)
```

### `next.config.js` Image Options

```js
module.exports = {
  images: {
    // v16 REQUIRED: restrict allowed quality values
    qualities: [25, 50, 75, 100],        // default: [75]

    // Allow remote images (use instead of deprecated `domains`)
    remotePatterns: [
      new URL('https://example.com/account/**'),
      // or object form:
      { protocol: 'https', hostname: 's3.amazonaws.com', pathname: '/my-bucket/**', search: '' },
      // wildcards: * = single segment, ** = any number (start/end only)
      { protocol: 'https', hostname: '**.example.com' },
    ],

    // Restrict local image paths
    localPatterns: [{ pathname: '/assets/images/**', search: '' }],

    // Custom loader file (project-root-relative)
    loader: 'custom',
    loaderFile: './my/image/loader.js',

    // Responsive breakpoints
    deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048, 3840],
    imageSizes: [32, 48, 64, 96, 128, 256, 384],

    // Format support
    formats: ['image/webp'],                    // default
    formats: ['image/avif', 'image/webp'],      // AVIF preferred, WebP fallback

    // Caching
    minimumCacheTTL: 14400,           // 4 hours (default)
    maximumDiskCacheSize: 500_000_000, // 500 MB

    // Redirects when fetching remote images
    maximumRedirects: 3,              // default

    // Performance / size limits
    maximumResponseBody: 50_000_000,  // 50 MB default

    // SVG (use with caution)
    dangerouslyAllowSVG: true,
    contentDispositionType: 'attachment',
    contentSecurityPolicy: "default-src 'self'; script-src 'none'; sandbox;",

    // Disable static imports
    disableStaticImages: true,

    // Local network images (private deployments only)
    dangerouslyAllowLocalIP: true,
  },
}
```

### Common Patterns

```tsx
// Responsive full-width from static import
import mountains from '../public/mountains.jpg'
<Image src={mountains} alt="Mountains" sizes="100vw" style={{ width: '100%', height: 'auto' }} />

// Remote image (must provide width/height for aspect ratio)
<Image src={photoUrl} alt="..." sizes="100vw" style={{ width: '100%', height: 'auto' }} width={500} height={300} />

// Background fill in a container
<div style={{ position: 'relative', width: '400px', height: '300px' }}>
  <Image fill src={url} alt="..." sizes="(min-width: 808px) 50vw, 100vw" style={{ objectFit: 'cover' }} />
</div>

// Light/dark theme images (CSS media query approach — avoids loading both)
// Use CSS to hide .imgDark by default, show it in dark mode
// Do NOT use preload/loading="eager" — it loads both images
```

### `priority` Deprecation (v16)
```tsx
// ❌ v15 and below
<Image priority src="..." />

// ✅ v16+
<Image preload src="..." />
```

---

## `next/link` — Client-Side Navigation

```tsx
import Link from 'next/link'

<Link href="/dashboard">Dashboard</Link>
```

### Props

| Prop | Default | Notes |
|---|---|---|
| `href` | — | Required. String or `{ pathname, query }` object |
| `replace` | `false` | Replace history entry instead of push |
| `scroll` | `true` | Scroll to top of page on navigation |
| `prefetch` | `"auto"` | Prefetch behavior (see below) |
| `onNavigate` | — | Callback for client-side nav only |
| `transitionTypes` | — | Passed to `React.addTransitionType` for `<ViewTransition>` |

All standard `<a>` attributes (`className`, `target`, `rel`, etc.) are forwarded.

### `prefetch` Values
- `"auto"` / `null` (default): Static routes → full prefetch; dynamic routes → prefetch to nearest `loading.js` boundary
- `true`: Full prefetch for all routes
- `false`: No prefetching

> Prefetching only runs in production.

### `onNavigate` vs `onClick`
- `onClick`: fires on every click
- `onNavigate`: fires **only during SPA navigation** — NOT for modifier+click (new tab), external URLs, or `download` links

```tsx
<Link
  href="/dashboard"
  onNavigate={(e) => {
    if (unsavedChanges && !confirm('Leave without saving?')) {
      e.preventDefault()  // blocks the navigation
    }
  }}
>
  Dashboard
</Link>
```

### `transitionTypes` (React View Transitions)
```tsx
<Link href="/about" transitionTypes={['slide-in']}>About</Link>
```

### Common Patterns

```tsx
// Object href with query params
<Link href={{ pathname: '/about', query: { name: 'test' } }}>About</Link>

// Hash/anchor scroll
<Link href="/dashboard#settings">Settings</Link>

// Dynamic routes from array
{posts.map(post => (
  <Link key={post.id} href={`/blog/${post.slug}`}>{post.title}</Link>
))}

// Active link detection
'use client'
import { usePathname } from 'next/navigation'
const pathname = usePathname()
<Link className={pathname === '/about' ? 'active' : ''} href="/about">About</Link>

// No scroll-to-top on navigation
<Link href="/page" scroll={false}>Page</Link>

// Replace instead of push
<Link href="/about" replace>About</Link>

// Proxy rewrite — tell Link both the display URL and real URL
<Link as="/dashboard" href={isAuthed ? '/auth/dashboard' : '/public/dashboard'}>
  Dashboard
</Link>
```

### Sticky Header Scroll Offset
When a sticky header clips scroll targets, add to global CSS:
```css
html {
  scroll-padding-top: 64px; /* height of your sticky header */
}
```

---

## `next/font` — Font Optimization

Self-hosts fonts at build time — no browser requests to Google.

### Google Fonts

```tsx
// app/layout.tsx
import { Inter, Roboto_Mono } from 'next/font/google'

// Variable font — no weight needed
const inter = Inter({
  subsets: ['latin'],
  display: 'swap',
})

// Non-variable font — weight required
const roboto = Roboto({ weight: '400', subsets: ['latin'], display: 'swap' })

// Multiple weights/styles
const roboto = Roboto({ weight: ['400', '700'], style: ['normal', 'italic'], subsets: ['latin'], display: 'swap' })

export default function RootLayout({ children }) {
  return (
    <html lang="en" className={inter.className}>
      <body>{children}</body>
    </html>
  )
}
```

> Font names with spaces use underscores: `Roboto Mono` → `Roboto_Mono`

### Local Fonts

```tsx
import localFont from 'next/font/local'

// Single file
const myFont = localFont({ src: './my-font.woff2', display: 'swap' })

// Multiple files for one family
const roboto = localFont({
  src: [
    { path: './Roboto-Regular.woff2', weight: '400', style: 'normal' },
    { path: './Roboto-Italic.woff2', weight: '400', style: 'italic' },
    { path: './Roboto-Bold.woff2', weight: '700', style: 'normal' },
    { path: './Roboto-BoldItalic.woff2', weight: '700', style: 'italic' },
  ],
})
```

### All Options

| Option | `google` | `local` | Type | Notes |
|---|---|---|---|---|
| `src` | — | ✓ | String or Array | Required for local |
| `weight` | ✓ | ✓ | String or Array | Required if not variable |
| `style` | ✓ | ✓ | String or Array | Default `'normal'` |
| `subsets` | ✓ | — | String[] | Needed for preloading |
| `axes` | ✓ | — | String[] | Extra variable font axes |
| `display` | ✓ | ✓ | String | Default `'swap'` |
| `preload` | ✓ | ✓ | Boolean | Default `true` |
| `fallback` | ✓ | ✓ | String[] | Fallback fonts |
| `adjustFontFallback` | ✓ | ✓ | Boolean or String | Reduces CLS; google default `true`, local default `'Arial'` |
| `variable` | ✓ | ✓ | String | CSS variable name e.g. `'--font-inter'` |
| `declarations` | — | ✓ | Object[] | Extra `@font-face` descriptors |

### Applying Fonts — 3 Methods

```tsx
// 1. className
<p className={inter.className}>Hello</p>

// 2. style object
<p style={inter.style}>Hello</p>

// 3. CSS variable (most flexible — use with external CSS or Tailwind)
const inter = Inter({ variable: '--font-inter', subsets: ['latin'] })
// Apply variable to parent, use in child via CSS
<html className={inter.variable}>
  ...
  <p className={styles.text}>Hello</p>  // .text { font-family: var(--font-inter) }
</html>
```

### With Tailwind CSS

```tsx
// app/layout.tsx
const inter = Inter({ subsets: ['latin'], variable: '--font-inter', display: 'swap' })
const roboto_mono = Roboto_Mono({ subsets: ['latin'], variable: '--font-roboto-mono', display: 'swap' })

<html className={`${inter.variable} ${roboto_mono.variable} antialiased`}>
```

```css
/* global.css — Tailwind v4 */
@import 'tailwindcss';
@theme inline {
  --font-sans: var(--font-inter);
  --font-mono: var(--font-roboto-mono);
}
```

```js
// tailwind.config.js — Tailwind v3
theme: {
  extend: {
    fontFamily: {
      sans: ['var(--font-inter)'],
      mono: ['var(--font-roboto-mono)'],
    },
  },
}
```

### Multiple Fonts — Shared Definitions File

```ts
// styles/fonts.ts — define once, import everywhere
import { Inter, Lora } from 'next/font/google'
import localFont from 'next/font/local'

export const inter = Inter({ subsets: ['latin'], display: 'swap' })
export const lora = Lora({ subsets: ['latin'], display: 'swap' })
export const greatVibes = localFont({ src: './GreatVibes-Regular.ttf' })
```

Add a path alias for convenience:
```json
// tsconfig.json
{ "compilerOptions": { "paths": { "@/fonts": ["./styles/fonts"] } } }
```

### Preloading Scope
- Used in `page.tsx` → preloaded on that route only
- Used in `layout.tsx` → preloaded on all routes in that layout
- Used in root `layout.tsx` → preloaded on **all routes**

---

## `next/form` — Form with Client Navigation

Extends HTML `<form>` with prefetching, client-side navigation, and progressive enhancement. Best for search/filter forms that update URL search params.

```tsx
import Form from 'next/form'

<Form action="/search">
  <input name="query" />
  <button type="submit">Search</button>
</Form>
// Submits as GET → navigates to /search?query=abc
```

### Two Modes

**`action` = string (GET navigation)**
```tsx
// Props: action (required), replace, scroll, prefetch
<Form action="/search" replace={false} scroll={true} prefetch={true}>
  <input name="query" />
  <button type="submit">Submit</button>
</Form>
```
- Encodes form data as URL search params
- Prefetches shared UI (`layout.js`, `loading.js`) when form enters viewport
- Client-side navigation on submit (no full page reload)
- Empty string `""` → same route, updated params

**`action` = function (Server Action mutation)**
```tsx
<Form action={createPost}>
  <input name="title" />
  <button type="submit">Create</button>
</Form>
```
- Behaves like a React form
- Cannot prefetch (destination unknown until action runs)

### Reading Search Params on Results Page

```tsx
// app/search/page.tsx
export default async function SearchPage({
  searchParams,
}: {
  searchParams: Promise<{ [key: string]: string | string[] | undefined }>
}) {
  const results = await getResults((await searchParams).query)
  return <div>...</div>
}
```

### Pending State with `useFormStatus`

```tsx
// app/ui/search-button.tsx
'use client'
import { useFormStatus } from 'react-dom'

export default function SearchButton() {
  const { pending } = useFormStatus()
  return <button type="submit">{pending ? 'Searching...' : 'Search'}</button>
}
```

### Server Action + Redirect Pattern

```tsx
// app/posts/actions.ts
'use server'
import { redirect } from 'next/navigation'

export async function createPost(formData: FormData) {
  const post = await db.posts.create({ title: formData.get('title') })
  redirect(`/posts/${post.id}`)
}
```

### Caveats
- `method`, `encType`, `target` are **not supported** — use plain `<form>` if needed
- `formAction` on buttons works but **doesn't prefetch**
- `<input type="file">` with string action submits filename only (browser behavior)
- `key` prop not supported with string `action`
- `onSubmit` + `event.preventDefault()` overrides `<Form>` behavior

---

## `next/script` — Third-Party Scripts

```tsx
import Script from 'next/script'

<Script src="https://example.com/script.js" />
```

### Props

| Prop | Type | Notes |
|---|---|---|
| `src` | String | Required unless inline script |
| `strategy` | String | Load timing (see below) |
| `onLoad` | Function | After script loads (Client Component only) |
| `onReady` | Function | After load + on every remount (Client Component only) |
| `onError` | Function | On load failure (Client Component only) |

### Loading Strategies

| Strategy | When | Use For |
|---|---|---|
| `afterInteractive` | **Default** — after some hydration | Analytics, tag managers |
| `beforeInteractive` | Before Next.js code, before hydration | Bot detectors, cookie consent |
| `lazyOnload` | Browser idle time | Chat widgets, social embeds |
| `worker` | Web worker (experimental, Pages Router only) | Offload heavy scripts from main thread |

```tsx
// beforeInteractive — must be in root layout, always in <head>
import Script from 'next/script'
export default function RootLayout({ children }) {
  return (
    <html><body>
      {children}
      <Script src="https://cdn.example.com/critical.js" strategy="beforeInteractive" />
    </body></html>
  )
}

// lazyOnload — low priority scripts
<Script src="https://widget.example.com/chat.js" strategy="lazyOnload" />

// worker — requires next.config.js flag + Pages Router only
// next.config.js: experimental: { nextScriptWorkers: true }
<Script src="https://example.com/heavy.js" strategy="worker" />
```

### `onLoad` — Run code after script loads
```tsx
'use client'
import Script from 'next/script'

export default function Page() {
  return (
    <Script
      src="https://cdnjs.cloudflare.com/ajax/libs/lodash.js/4.17.20/lodash.min.js"
      onLoad={() => console.log(_.sample([1, 2, 3, 4]))}
    />
  )
}
```
> Cannot be used with `beforeInteractive`.

### `onReady` — Run after load AND on every remount
```tsx
'use client'
import { useRef } from 'react'
import Script from 'next/script'

export default function Page() {
  const mapRef = useRef()
  return (
    <>
      <div ref={mapRef} />
      <Script
        id="google-maps"
        src="https://maps.googleapis.com/maps/api/js"
        onReady={() => {
          new google.maps.Map(mapRef.current, {
            center: { lat: -34.397, lng: 150.644 },
            zoom: 8,
          })
        }}
      />
    </>
  )
}
```

### `onError` — Handle load failures
```tsx
'use client'
<Script
  src="https://example.com/script.js"
  onError={(e) => console.error('Script failed to load', e)}
/>
```
> Cannot be used with `beforeInteractive`.

### Key Rules
- `beforeInteractive` → root `layout.tsx` only
- `onLoad`, `onReady`, `onError` → Client Components only
- `worker` → Pages Router only, requires `experimental.nextScriptWorkers: true`
- `id` prop required for inline scripts (for deduplication)