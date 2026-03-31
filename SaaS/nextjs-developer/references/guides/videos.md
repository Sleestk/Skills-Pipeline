# Videos

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/videos

## `<video>` tag (self-hosted)

```jsx
export function Video() {
  return (
    <video width="320" height="240" controls preload="none">
      <source src="/path/to/video.mp4" type="video/mp4" />
      <track src="/path/to/captions.vtt" kind="subtitles" srcLang="en" label="English" />
      Your browser does not support the video tag.
    </video>
  )
}
```

Key attributes: `src`, `width`, `height`, `controls`, `autoPlay`, `loop`, `muted`, `preload` (`none`/`metadata`/`auto`), `playsInline`

> For `autoPlay` to work in most browsers: add `muted` and `playsInline`.

**Best practices:** Include fallback content, add subtitles via `<track>`, use standard HTML5 controls for accessibility.

## `<iframe>` (external platforms)

```jsx
<iframe src="https://www.youtube.com/embed/19g66ezsKAg" allowFullScreen />
```

Key attributes: `src`, `width`, `height`, `allowFullScreen`, `sandbox`, `loading="lazy"`, `title`

## Embedding Externally Hosted Videos

```jsx
// app/ui/video-component.jsx — Server Component
export default async function VideoComponent() {
  const src = await getVideoSrc()
  return <iframe src={src} allowFullScreen />
}

// app/page.jsx — stream with Suspense
import { Suspense } from 'react'
import VideoComponent from '../ui/VideoComponent'
export default function Page() {
  return (
    <Suspense fallback={<VideoSkeleton />}>
      <VideoComponent />
    </Suspense>
  )
}
```

## Self-Hosting with Vercel Blob

```jsx
// Upload via Vercel dashboard or server-side upload API
import { list } from '@vercel/blob'

async function VideoComponent({ fileName }) {
  const { blobs } = await list({ prefix: fileName, limit: 1 })
  return (
    <video controls preload="none" aria-label="Video player">
      <source src={blobs[0].url} type="video/mp4" />
    </video>
  )
}
```

## Resources

- **Formats/codecs**: [Mozilla video codecs guide](https://developer.mozilla.org/en-US/docs/Web/Media/Formats/Video_codecs)
- **Compression**: [FFmpeg](https://www.ffmpeg.org/)
- **Community `next-video` component** — supports Vercel Blob, S3, Backblaze, Mux: https://next-video.dev/docs
- **Cloudinary integration**: https://next.cloudinary.dev/
- **Mux**: https://www.mux.com/for/nextjs
- **ImageKit.io**: https://imagekit.io/docs/integration/nextjs