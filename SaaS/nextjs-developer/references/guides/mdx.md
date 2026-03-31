# MDX

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/mdx

MDX = Markdown + JSX. Write markdown with React components inline.

## Install

```bash
pnpm add @next/mdx @mdx-js/loader @mdx-js/react @types/mdx
```

## Configure `next.config.mjs`

```js
import createMDX from '@next/mdx'

const nextConfig = {
  pageExtensions: ['js', 'jsx', 'md', 'mdx', 'ts', 'tsx'],
}

const withMDX = createMDX({
  // Add remark/rehype plugins here
})

export default withMDX(nextConfig)
```

To handle `.md` files too:
```js
const withMDX = createMDX({ extension: /\.(md|mdx)$/ })
```

## `mdx-components.tsx` (required for App Router)

```tsx
// mdx-components.tsx (project root)
import type { MDXComponents } from 'mdx/types'

const components: MDXComponents = {}
export function useMDXComponents(): MDXComponents {
  return components
}
```

## Rendering MDX

### File-based routing
Create `app/mdx-page/page.mdx` — imports React components directly:
```mdx
import { MyComponent } from 'my-component'
# Welcome!
<MyComponent />
```

### Import into a page
```tsx
import Welcome from '@/markdown/welcome.mdx'
export default function Page() { return <Welcome /> }
```

### Dynamic imports
```tsx
export default async function Page({ params }) {
  const { slug } = await params
  const { default: Post } = await import(`@/content/${slug}.mdx`)
  return <Post />
}
export function generateStaticParams() { return [{ slug: 'welcome' }] }
export const dynamicParams = false
```

## Custom Styles and Components

### Global (affects all MDX)
```tsx
// mdx-components.tsx
const components = {
  h1: ({ children }) => <h1 style={{ color: 'red', fontSize: '48px' }}>{children}</h1>,
  img: (props) => <Image sizes="100vw" style={{ width: '100%', height: 'auto' }} {...props} />,
}
export function useMDXComponents(): MDXComponents { return components }
```

### Local (per page)
```tsx
const overrideComponents = { h1: CustomH1 }
return <Welcome components={overrideComponents} />
```

### Shared layouts
Use `app/mdx-page/layout.tsx` — normal Next.js layout convention.

### Tailwind typography
```tsx
<div className="prose prose-headings:mt-8 prose-headings:font-semibold ...">
  {children}
</div>
```

## Frontmatter

`@next/mdx` doesn't support frontmatter by default. Use `remark-frontmatter` or `gray-matter`. Alternatively, use named exports:

```mdx
export const metadata = { author: 'John Doe' }
# Blog post
```

```tsx
import BlogPost, { metadata } from '@/content/blog-post.mdx'
```

## remark/rehype Plugins

```js
// next.config.mjs
import remarkGfm from 'remark-gfm'
const withMDX = createMDX({ options: { remarkPlugins: [remarkGfm], rehypePlugins: [] } })
```

With Turbopack, specify plugins as strings:
```js
const withMDX = createMDX({
  options: {
    remarkPlugins: ['remark-gfm', ['remark-toc', { heading: 'The Table' }]],
    rehypePlugins: ['rehype-slug'],
  },
})
```

## Experimental Rust-based Compiler

```js
module.exports = withMDX({ experimental: { mdxRs: true } })
// or with options:
// mdxRs: { jsxRuntime?, jsxImportSource?, mdxType?: 'gfm' | 'commonmark' }
```