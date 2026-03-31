# Next.js Integration

## Packages

```bash
npm install @supabase/supabase-js @supabase/ssr
```

## Environment Variables

```bash
# .env.local
NEXT_PUBLIC_SUPABASE_URL=your-project-url
NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY=sb_publishable_... or anon key
```

Get from the **Connect** dialog or **API Keys** in your project settings.

## Two Client Types

Next.js needs two Supabase clients:

1. **Browser client** — for Client Components (singleton)
2. **Server client** — for Server Components, Server Actions, Route Handlers (new per request)

## Utility Functions

Create `lib/supabase/` with two files:

### Browser client — `lib/supabase/client.ts`
```ts
import { createBrowserClient } from '@supabase/ssr'

export function createClient() {
  return createBrowserClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY!
  )
}
```
`createBrowserClient` is a singleton — calling it multiple times returns the same instance.

### Server client — `lib/supabase/server.ts`
```ts
import { createServerClient } from '@supabase/ssr'
import { cookies } from 'next/headers'

export async function createClient() {
  const cookieStore = await cookies()

  return createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY!,
    {
      cookies: {
        getAll() {
          return cookieStore.getAll()
        },
        setAll(cookiesToSet) {
          try {
            cookiesToSet.forEach(({ name, value, options }) =>
              cookieStore.set(name, value, options)
            )
          } catch {
            // The `setAll` method is called from a Server Component.
            // This can be ignored if you have middleware refreshing sessions.
          }
        },
      },
    }
  )
}
```
**Create a new server client for every request.** It configures a `fetch` call using that request's cookies.

## Proxy (Middleware)

The Proxy refreshes expired auth tokens and writes updated cookies. Create `middleware.ts` (or `src/middleware.ts`):

```ts
import { createServerClient } from '@supabase/ssr'
import { NextResponse, type NextRequest } from 'next/server'

export async function middleware(request: NextRequest) {
  let supabaseResponse = NextResponse.next({ request })

  const supabase = createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY!,
    {
      cookies: {
        getAll() {
          return request.cookies.getAll()
        },
        setAll(cookiesToSet, headers) {
          cookiesToSet.forEach(({ name, value, options }) =>
            request.cookies.set(name, value)
          )
          supabaseResponse = NextResponse.next({ request })
          cookiesToSet.forEach(({ name, value, options }) =>
            supabaseResponse.cookies.set(name, value, options)
          )
          Object.entries(headers).forEach(([key, value]) =>
            supabaseResponse.headers.set(key, value)
          )
        },
      },
    }
  )

  // Refresh session — IMPORTANT
  await supabase.auth.getClaims()

  return supabaseResponse
}

export const config = {
  matcher: [
    '/((?!_next/static|_next/image|favicon.ico|.*\\.(?:svg|png|jpg|jpeg|gif|webp)$).*)',
  ],
}
```

### Critical rules:
- Use `supabase.auth.getClaims()` to refresh the session. **Never use `getSession()` in server code** — it doesn't revalidate the JWT.
- The middleware must pass refreshed cookies to both the request (for Server Components) and the response (for the browser).

## Auth Confirmation Route (PKCE)

For email confirmation and password reset, create `/app/auth/confirm/route.ts`:

```ts
import { type EmailOtpType } from '@supabase/supabase-js'
import { type NextRequest } from 'next/server'
import { createClient } from '@/lib/supabase/server'
import { redirect } from 'next/navigation'

export async function GET(request: NextRequest) {
  const { searchParams } = new URL(request.url)
  const token_hash = searchParams.get('token_hash')
  const type = searchParams.get('type') as EmailOtpType | null
  const next = searchParams.get('next') ?? '/'

  if (token_hash && type) {
    const supabase = await createClient()
    const { error } = await supabase.auth.verifyOtp({ type, token_hash })
    if (!error) {
      redirect(next)
    }
  }

  redirect('/auth/auth-code-error')
}
```

Update email templates to use `{{ .TokenHash }}` and point to this route.

## Protecting Pages

### Server Component
```ts
import { createClient } from '@/lib/supabase/server'
import { redirect } from 'next/navigation'

export default async function ProtectedPage() {
  const supabase = await createClient()
  const { data: { user } } = await supabase.auth.getUser()

  if (!user) {
    redirect('/login')
  }

  return <div>Hello {user.email}</div>
}
```

### Server Action
```ts
'use server'
import { createClient } from '@/lib/supabase/server'

export async function addTodo(formData: FormData) {
  const supabase = await createClient()
  const { data: { user } } = await supabase.auth.getUser()

  if (!user) throw new Error('Not authenticated')

  const { error } = await supabase
    .from('todos')
    .insert({ task: formData.get('task'), user_id: user.id })
}
```

### Client Component
```tsx
'use client'
import { createClient } from '@/lib/supabase/client'

export default function ClientComponent() {
  const supabase = createClient()

  // Use supabase.from(), supabase.auth.*, etc.
}
```

## Caching Considerations

- **Never enable ISR** on authenticated routes — cached `Set-Cookie` headers leak sessions
- Use `export const dynamic = 'force-dynamic'` on pages that handle auth
- `@supabase/ssr` v0.10.0+ automatically sets cache headers to prevent CDN caching of refreshed tokens

## Quick Start Template

```bash
npx create-next-app -e with-supabase
```

This gives you cookie-based auth, TypeScript, and Tailwind CSS pre-configured.