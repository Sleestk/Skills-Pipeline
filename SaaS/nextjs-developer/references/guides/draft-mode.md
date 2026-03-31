# Draft Mode

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/draft-mode

Draft Mode allows previewing headless CMS draft content on statically generated pages by switching to dynamic rendering on demand.

## Step 1: Route Handler to enable draft mode

```ts
// app/api/draft/route.ts
import { draftMode } from 'next/headers'
import { redirect } from 'next/navigation'

export async function GET(request: Request) {
  const { searchParams } = new URL(request.url)
  const secret = searchParams.get('secret')
  const slug = searchParams.get('slug')

  // Verify secret token
  if (secret !== 'MY_SECRET_TOKEN' || !slug) {
    return new Response('Invalid token', { status: 401 })
  }

  // Check if slug exists in CMS
  const post = await getPostBySlug(slug)
  if (!post) return new Response('Invalid slug', { status: 401 })

  // Enable Draft Mode (sets __prerender_bypass cookie)
  const draft = await draftMode()
  draft.enable()

  // Redirect to the post (not to searchParams.slug — open redirect risk)
  redirect(post.slug)
}
```

URL format: `https://<your-site>/api/draft?secret=<token>&slug=<path>`

Test manually by visiting `/api/draft` — check for `Set-Cookie: __prerender_bypass` in response.

## Step 2: Check `isEnabled` in the page

```tsx
// app/page.tsx
import { draftMode } from 'next/headers'

async function getData() {
  const { isEnabled } = await draftMode()
  const url = isEnabled ? 'https://draft.example.com' : 'https://production.example.com'
  const res = await fetch(url)
  return res.json()
}

export default async function Page() {
  const { title, desc } = await getData()
  return <main><h1>{title}</h1><p>{desc}</p></main>
}
```

When `isEnabled` is `true`, data is fetched at **request time** instead of build time, showing draft content.

Visit the Route Handler from your CMS to enable draft mode, then navigate to the page to see draft content.