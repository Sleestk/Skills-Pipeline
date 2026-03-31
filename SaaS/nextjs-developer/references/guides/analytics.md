# Analytics

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/analytics

## Client Instrumentation File

For global analytics / error tracking that runs before the app starts, create `instrumentation-client.js|ts` in your app root:

```js
// instrumentation-client.js
console.log('Analytics initialized')
window.addEventListener('error', (event) => {
  reportError(event.error)
})
```

## useReportWebVitals

Use the `useReportWebVitals` hook in a Client Component to report Web Vitals:

```jsx
// app/_components/web-vitals.js
'use client'
import { useReportWebVitals } from 'next/web-vitals'

export function WebVitals() {
  useReportWebVitals((metric) => {
    switch (metric.name) {
      case 'FCP': { /* handle FCP */ break }
      case 'LCP': { /* handle LCP */ break }
      case 'CLS': { /* handle CLS */ break }
      case 'FID': { /* handle FID */ break }
      case 'INP': { /* handle INP */ break }
      case 'TTFB': { /* handle TTFB */ break }
    }
  })
}
```

Import it in the root layout — keep it in a separate component to confine the `'use client'` boundary:
```jsx
// app/layout.js
import { WebVitals } from './_components/web-vitals'
export default function Layout({ children }) {
  return <html><body><WebVitals />{children}</body></html>
}
```

## Sending to external systems

```js
useReportWebVitals((metric) => {
  const body = JSON.stringify(metric)
  const url = 'https://example.com/analytics'
  if (navigator.sendBeacon) {
    navigator.sendBeacon(url, body)
  } else {
    fetch(url, { body, method: 'POST', keepalive: true })
  }
})
```

For Google Analytics, use `metric.id` to build distributions:
```js
useReportWebVitals((metric) => {
  window.gtag('event', metric.name, {
    value: Math.round(metric.name === 'CLS' ? metric.value * 1000 : metric.value),
    event_label: metric.id,
    non_interaction: true,
  })
})
```

## Web Vitals covered
- **TTFB** — Time to First Byte
- **FCP** — First Contentful Paint
- **LCP** — Largest Contentful Paint
- **FID** — First Input Delay
- **CLS** — Cumulative Layout Shift
- **INP** — Interaction to Next Paint