# Authentication

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/authentication

Three concepts: **Authentication** (who are you), **Session Management** (persisting auth state), **Authorization** (what can you access).

## Sign-up / Login with Server Actions

### 1. Capture credentials with a form
```tsx
// app/ui/signup-form.tsx
import { signup } from '@/app/actions/auth'
export function SignupForm() {
  return (
    <form action={signup}>
      <input name="name" /><input name="email" type="email" /><input name="password" type="password" />
      <button type="submit">Sign Up</button>
    </form>
  )
}
```

### 2. Validate with Zod
```ts
// app/lib/definitions.ts
import * as z from 'zod'
export const SignupFormSchema = z.object({
  name: z.string().min(2).trim(),
  email: z.email().trim(),
  password: z.string().min(8).regex(/[a-zA-Z]/).regex(/[0-9]/).regex(/[^a-zA-Z0-9]/).trim(),
})
```

### 3. Create user + session
```ts
// app/actions/auth.ts
export async function signup(state: FormState, formData: FormData) {
  const validatedFields = SignupFormSchema.safeParse(Object.fromEntries(formData))
  if (!validatedFields.success) return { errors: validatedFields.error.flatten().fieldErrors }
  const hashedPassword = await bcrypt.hash(validatedFields.data.password, 10)
  // Insert to DB...
  await createSession(user.id)
  redirect('/profile')
}
```

Use `useActionState` in Client Component to display validation errors.

## Session Management

### Stateless (JWT cookies)

```ts
// app/lib/session.ts
import 'server-only'
import { SignJWT, jwtVerify } from 'jose'
const secretKey = process.env.SESSION_SECRET
const encodedKey = new TextEncoder().encode(secretKey)

export async function encrypt(payload) {
  return new SignJWT(payload).setProtectedHeader({ alg: 'HS256' })
    .setIssuedAt().setExpirationTime('7d').sign(encodedKey)
}
export async function decrypt(session = '') {
  try {
    const { payload } = await jwtVerify(session, encodedKey, { algorithms: ['HS256'] })
    return payload
  } catch { console.log('Failed to verify session') }
}
export async function createSession(userId: string) {
  const expiresAt = new Date(Date.now() + 7 * 24 * 60 * 60 * 1000)
  const session = await encrypt({ userId, expiresAt })
  const cookieStore = await cookies()
  cookieStore.set('session', session, { httpOnly: true, secure: true, expires: expiresAt, sameSite: 'lax', path: '/' })
}
export async function deleteSession() {
  (await cookies()).delete('session')
}
```

Cookie options: **HttpOnly** (no JS access), **Secure** (HTTPS), **SameSite**, **Max-Age/Expires**, **Path**.

### Database sessions
Store session in DB, encrypt session ID in cookie, sync on each request.

## Authorization

### Proxy (optimistic checks only — cookie reads)
```ts
// proxy.ts
export default async function proxy(req: NextRequest) {
  const cookie = (await cookies()).get('session')?.value
  const session = await decrypt(cookie)
  if (isProtectedRoute && !session?.userId) {
    return NextResponse.redirect(new URL('/login', req.nextUrl))
  }
}
```
**Proxy should NOT be your only security layer** — it's for performance/UX, not security.

### Data Access Layer (DAL)
```ts
// app/lib/dal.ts
import 'server-only'
export const verifySession = cache(async () => {
  const session = await decrypt((await cookies()).get('session')?.value)
  if (!session?.userId) redirect('/login')
  return { isAuth: true, userId: session.userId }
})
```

Call `verifySession()` in data requests, Server Actions, Route Handlers.

### Authorization in Server Actions
```ts
'use server'
export async function serverAction() {
  const session = await verifySession()
  if (session?.user?.role !== 'admin') return null
  // proceed...
}
```

### Authorization in Route Handlers
```ts
export async function GET() {
  const session = await verifySession()
  if (!session) return new Response(null, { status: 401 })
  if (session.user.role !== 'admin') return new Response(null, { status: 403 })
}
```

## Auth Libraries
Auth0, Better Auth, Clerk, Descope, Kinde, Logto, NextAuth.js, Ory, Stack Auth, Supabase, Stytch, WorkOS

## Session Libraries
Iron Session, Jose