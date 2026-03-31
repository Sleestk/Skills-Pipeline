# JSON-LD

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/json-ld

Add structured data to help search engines and AI understand page content.

## Usage

Render as a `<script>` tag in `layout.js` or `page.js`. Replace `<` with `\u003c` to prevent XSS:

```tsx
// app/products/[id]/page.tsx
export default async function Page({ params }) {
  const { id } = await params
  const product = await getProduct(id)

  const jsonLd = {
    '@context': 'https://schema.org',
    '@type': 'Product',
    name: product.name,
    image: product.image,
    description: product.description,
  }

  return (
    <section>
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{
          __html: JSON.stringify(jsonLd).replace(/</g, '\\u003c'),
        }}
      />
      {/* rest of page */}
    </section>
  )
}
```

> Use a native `<script>` tag (not `next/script`) — JSON-LD is structured data, not executable code.

## TypeScript types

```tsx
import { Product, WithContext } from 'schema-dts'

const jsonLd: WithContext<Product> = {
  '@context': 'https://schema.org',
  '@type': 'Product',
  name: 'Next.js Sticker',
  image: 'https://nextjs.org/imgs/sticker.png',
  description: 'Dynamic at the speed of static.',
}
```

Install: `npm install schema-dts`

## Validation

- [Google Rich Results Test](https://search.google.com/test/rich-results)
- [Schema Markup Validator](https://validator.schema.org/)