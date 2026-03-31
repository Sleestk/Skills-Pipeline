# Scripts

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/scripts

Use `next/script` to optimize third-party script loading.

## Loading Strategies

```tsx
import Script from 'next/script'
```

| Strategy | When loaded | Use case |
|---|---|---|
| `beforeInteractive` | Before Next.js code + before hydration | Critical scripts that must load first |
| `afterInteractive` (default) | Early but after some hydration | Analytics, tag managers |
| `lazyOnload` | Browser idle time | Low-priority scripts |
| `worker` (experimental) | Web worker via Partytown | Offload main thread |

## Layout Scripts (multiple routes)
```tsx
// app/dashboard/layout.tsx
import Script from 'next/script'
export default function Layout({ children }) {
  return (
    <>
      <section>{children}</section>
      <Script src="https://example.com/script.js" />
    </>
  )
}
```
Script loads once even if user navigates between routes in the same layout.

## Application Scripts (all routes)
```tsx
// app/layout.tsx
<Script src="https://example.com/script.js" />
```

## Inline Scripts
```tsx
<Script id="show-banner">
  {`document.getElementById('banner').classList.remove('hidden')`}
</Script>
// or:
<Script id="show-banner" dangerouslySetInnerHTML={{ __html: `...` }} />
```
`id` is required for inline scripts.

## Event Handlers (Client Components only)
```tsx
'use client'
import Script from 'next/script'
<Script src="https://example.com/script.js" onLoad={() => console.log('loaded')} />
```
Events: `onLoad`, `onReady`, `onError`.

## Web Worker (experimental)
```js
// next.config.js
module.exports = { experimental: { nextScriptWorkers: true } }
```
```bash
npm install @qwik.dev/partytown
```
```tsx
<Script src="..." strategy="worker" />
```
Offloads script to Partytown web worker. Trade-offs: DOM access is async, some APIs may not work.

## Additional Attributes
Any extra props (e.g. `nonce`, `data-test`) are forwarded to the final `<script>` tag.