# Internationalization

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/internationalization

## Routing with Proxy

Detect locale from `Accept-Language` header and redirect in `proxy.js`:

```js
// proxy.js
import { match } from '@formatjs/intl-localematcher'
import Negotiator from 'negotiator'
import { NextResponse } from 'next/server'

const locales = ['en-US', 'nl-NL', 'nl']

function getLocale(request) {
  const headers = { 'accept-language': request.headers.get('accept-language') ?? '' }
  const languages = new Negotiator({ headers }).languages()
  return match(languages, locales, 'en-US')
}

export function proxy(request) {
  const { pathname } = request.nextUrl
  const pathnameHasLocale = locales.some(
    (locale) => pathname.startsWith(`/${locale}/`) || pathname === `/${locale}`
  )
  if (pathnameHasLocale) return

  const locale = getLocale(request)
  request.nextUrl.pathname = `/${locale}${pathname}`
  return NextResponse.redirect(request.nextUrl)
}

export const config = { matcher: ['/((?!_next).*)'] }
```

## App Directory Structure

All special files nested under `app/[lang]`:

```tsx
// app/[lang]/page.tsx
export default async function Page({ params }: PageProps<'/[lang]'>) {
  const { lang } = await params
  return ...
}
```

`PageProps` and `LayoutProps` are globally available TypeScript helpers for route params.

## Localization (Dictionaries)

```json
// dictionaries/en.json
{ "products": { "cart": "Add to Cart" } }
// dictionaries/nl.json
{ "products": { "cart": "Toevoegen aan Winkelwagen" } }
```

```ts
// app/[lang]/dictionaries.ts
import 'server-only'
const dictionaries = {
  en: () => import('./dictionaries/en.json').then(m => m.default),
  nl: () => import('./dictionaries/nl.json').then(m => m.default),
}
export type Locale = keyof typeof dictionaries
export const hasLocale = (locale: string): locale is Locale => locale in dictionaries
export const getDictionary = async (locale: Locale) => dictionaries[locale]()
```

```tsx
// app/[lang]/page.tsx
import { notFound } from 'next/navigation'
import { getDictionary, hasLocale } from './dictionaries'

export default async function Page({ params }: PageProps<'/[lang]'>) {
  const { lang } = await params
  if (!hasLocale(lang)) notFound()
  const dict = await getDictionary(lang)
  return <button>{dict.products.cart}</button>
}
```

Using `hasLocale` narrows the type and returns 404 for missing translations instead of a runtime error.

## Static Rendering

```tsx
// app/[lang]/layout.tsx
export async function generateStaticParams() {
  return [{ lang: 'en-US' }, { lang: 'de' }]
}
export default async function RootLayout({ children, params }) {
  return <html lang={(await params).lang}><body>{children}</body></html>
}
```

## Libraries
`next-intl`, `next-international`, `next-i18n-router`, `paraglide-next`, `lingui`, `tolgee`, `next-intlayer`, `gt-next`