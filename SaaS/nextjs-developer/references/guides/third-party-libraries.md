# Third-Party Libraries

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/third-party-libraries

`@next/third-parties` provides optimized components for popular libraries.

```bash
npm install @next/third-parties@latest next@latest
```

## Google Tag Manager

```tsx
// app/layout.tsx (all routes)
import { GoogleTagManager } from '@next/third-parties/google'
export default function RootLayout({ children }) {
  return <html><GoogleTagManager gtmId="GTM-XYZ" /><body>{children}</body></html>
}
```

**Send events:**
```tsx
'use client'
import { sendGTMEvent } from '@next/third-parties/google'
<button onClick={() => sendGTMEvent({ event: 'buttonClicked', value: 'xyz' })}>Send</button>
```

Options: `gtmId`*, `gtmScriptUrl`, `dataLayer`, `dataLayerName`, `auth`, `preview`

## Google Analytics (GA4)

```tsx
import { GoogleAnalytics } from '@next/third-parties/google'
<GoogleAnalytics gaId="G-XYZ" />
```

**Send events:**
```tsx
'use client'
import { sendGAEvent } from '@next/third-parties/google'
<button onClick={() => sendGAEvent('event', 'buttonClicked', { value: 'xyz' })}>Send</button>
```

Tracks pageviews automatically via browser history. Requires "Enhanced Measurement" enabled in GA admin + "Page changes based on browser history events" checked.

Options: `gaId`*, `dataLayerName`, `nonce`

> If already using GTM, configure GA4 through GTM instead.

## Google Maps Embed

```jsx
import { GoogleMapsEmbed } from '@next/third-parties/google'
<GoogleMapsEmbed apiKey="XYZ" height={200} width="100%" mode="place" q="Brooklyn+Bridge,New+York,NY" />
```

Lazy-loaded by default. Options: `apiKey`*, `mode`*, `height`, `width`, `style`, `allowfullscreen`, `loading`, `q`, `center`, `zoom`, `maptype`, `language`, `region`

## YouTube Embed

```jsx
import { YouTubeEmbed } from '@next/third-parties/google'
<YouTubeEmbed videoid="ogfYd705cRs" height={400} params="controls=0" />
```

Uses `lite-youtube-embed` for faster loading. Options: `videoid`*, `width`, `height`, `playlabel`, `params`, `style`