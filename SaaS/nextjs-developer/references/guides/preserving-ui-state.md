# Preserving UI State

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/preserving-ui-state
> Requires `cacheComponents: true` in `next.config.js`

Next.js uses React's `<Activity>` component to hide (not unmount) pages on navigation, preserving DOM state, React state, scroll position, form values, video playback position.

Preserves up to **3 routes**. Beyond that, oldest is evicted and re-renders fresh.

## Resetting State on Navigation

### Dropdown / transient open state
```tsx
useLayoutEffect(() => {
  return () => setIsOpen(false)  // cleanup runs when Activity hides component
}, [])
```

### Form inputs (reset after submit)
```tsx
<form ref={(form) => {
  return () => form?.reset()  // callback ref cleanup
}}>
```

### `useActionState` reset after action
Add a `RESET` action type and dispatch it in `useLayoutEffect` cleanup when `shouldReset.current` is true.

### Dialog initialization (focus, etc.)
Derive dialog state from URL (`searchParams`) instead of component state — so it resets when URL changes on navigation.

## Global Styles

Disable page-level CSS variables/classes when hidden:
```tsx
<style ref={(style) => {
  if (style) style.media = ''           // enable when visible
  return () => { if (style) style.media = 'not all' }  // disable when hidden
}}>
  {`:root { --page-accent: blue; }`}
</style>
```

## Using `<Activity>` Directly

Useful for tabs, accordions — hide content without unmounting:

```tsx
'use client'
import { Activity, Suspense, useState } from 'react'

export function ExpandableComments({ commentsPromise }) {
  const [expanded, setExpanded] = useState(false)
  return (
    <>
      <button onClick={() => setExpanded(e => !e)}>Toggle Comments</button>
      <Activity mode={expanded ? 'visible' : 'hidden'}>
        <Suspense fallback={<div>Loading...</div>}>
          <Comments commentsPromise={commentsPromise} />
        </Suspense>
      </Activity>
    </>
  )
}
```

Hidden `<Activity>` content renders at lower priority — data starts fetching before user reveals it.

## Effects and Media Cleanup

Effects (timers, subscriptions) run cleanup when Activity hides, then re-run when visible again. For media elements (`<video>`, `<audio>`), `display: none` doesn't pause playback — pause explicitly:

```tsx
useLayoutEffect(() => {
  const video = videoRef.current
  return () => { video?.pause() }  // pauses on hide, preserves position
}, [])
```

## First Mount vs Re-show

```tsx
const hasMountedRef = useRef(false)
useEffect(() => {
  if (!hasMountedRef.current) {
    hasMountedRef.current = true
    console.log('First mount')
  } else {
    console.log('Became visible again')
  }
}, [])
```

## Testing with Activity

`display: none` content stays in DOM. Use visibility-aware selectors:
```ts
// Playwright — getByRole automatically filters by visibility
await page.getByRole('button', { name: 'Submit' }).click()
// Explicit fallback:
await page.locator('.card').filter({ visible: true }).first().click()
```