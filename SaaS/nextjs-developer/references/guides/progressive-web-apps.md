# Progressive Web Apps (PWAs)

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/progressive-web-apps

## 1. Web App Manifest

```ts
// app/manifest.ts
import type { MetadataRoute } from 'next'
export default function manifest(): MetadataRoute.Manifest {
  return {
    name: 'Next.js PWA',
    short_name: 'NextPWA',
    description: 'A Progressive Web App built with Next.js',
    start_url: '/',
    display: 'standalone',
    background_color: '#ffffff',
    theme_color: '#000000',
    icons: [
      { src: '/icon-192x192.png', sizes: '192x192', type: 'image/png' },
      { src: '/icon-512x512.png', sizes: '512x512', type: 'image/png' },
    ],
  }
}
```

## 2. Web Push Notifications

### Install VAPID keys
```bash
npm install -g web-push
web-push generate-vapid-keys
# Add to .env:
# NEXT_PUBLIC_VAPID_PUBLIC_KEY=...
# VAPID_PRIVATE_KEY=...
```

### Subscribe/unsubscribe/send (Server Actions)
```ts
// app/actions.ts
'use server'
import webpush from 'web-push'
webpush.setVapidDetails('mailto:your-email@example.com',
  process.env.NEXT_PUBLIC_VAPID_PUBLIC_KEY!,
  process.env.VAPID_PRIVATE_KEY!)

let subscription: PushSubscription | null = null

export async function subscribeUser(sub: PushSubscription) {
  subscription = sub  // store in DB in production
  return { success: true }
}
export async function unsubscribeUser() {
  subscription = null
  return { success: true }
}
export async function sendNotification(message: string) {
  if (!subscription) throw new Error('No subscription available')
  await webpush.sendNotification(subscription, JSON.stringify({
    title: 'Test Notification', body: message, icon: '/icon.png',
  }))
  return { success: true }
}
```

### Client Component
```tsx
'use client'
import { useEffect, useState } from 'react'
import { subscribeUser, unsubscribeUser, sendNotification } from './actions'

function PushNotificationManager() {
  const [isSupported, setIsSupported] = useState(false)
  const [subscription, setSubscription] = useState<PushSubscription | null>(null)

  useEffect(() => {
    if ('serviceWorker' in navigator && 'PushManager' in window) {
      setIsSupported(true)
      navigator.serviceWorker.register('/sw.js', { scope: '/', updateViaCache: 'none' })
        .then(reg => reg.pushManager.getSubscription().then(setSubscription))
    }
  }, [])

  async function subscribe() {
    const reg = await navigator.serviceWorker.ready
    const sub = await reg.pushManager.subscribe({
      userVisibleOnly: true,
      applicationServerKey: urlBase64ToUint8Array(process.env.NEXT_PUBLIC_VAPID_PUBLIC_KEY!),
    })
    setSubscription(sub)
    await subscribeUser(JSON.parse(JSON.stringify(sub)))
  }

  if (!isSupported) return <p>Push notifications not supported.</p>
  return subscription
    ? <button onClick={async () => { await subscription.unsubscribe(); setSubscription(null); await unsubscribeUser() }}>Unsubscribe</button>
    : <button onClick={subscribe}>Subscribe</button>
}
```

## 3. Service Worker (`public/sw.js`)
```js
self.addEventListener('push', function(event) {
  if (event.data) {
    const data = event.data.json()
    event.waitUntil(self.registration.showNotification(data.title, {
      body: data.body, icon: data.icon || '/icon.png', badge: '/badge.png',
    }))
  }
})
self.addEventListener('notificationclick', function(event) {
  event.notification.close()
  event.waitUntil(clients.openWindow('https://your-website.com'))
})
```

## 4. iOS Install Prompt

```tsx
function InstallPrompt() {
  const [isIOS, setIsIOS] = useState(false)
  const [isStandalone, setIsStandalone] = useState(false)
  useEffect(() => {
    setIsIOS(/iPad|iPhone|iPod/.test(navigator.userAgent))
    setIsStandalone(window.matchMedia('(display-mode: standalone)').matches)
  }, [])
  if (isStandalone) return null
  return isIOS ? <p>Tap share ⎋ then "Add to Home Screen" ➕</p> : <button>Add to Home Screen</button>
}
```

## Testing Locally
- Use `next dev --experimental-https`
- Browser notifications must be enabled
- Requires HTTPS even in development for service workers

## Security Headers for Service Worker
```js
{ source: '/sw.js', headers: [
  { key: 'Content-Type', value: 'application/javascript; charset=utf-8' },
  { key: 'Cache-Control', value: 'no-cache, no-store, must-revalidate' },
  { key: 'Content-Security-Policy', value: "default-src 'self'; script-src 'self'" },
]}
```

## Offline Support
Use [Serwist](https://github.com/serwist/serwist) with Next.js for offline functionality (requires webpack config).