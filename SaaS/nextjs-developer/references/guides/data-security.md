# Data Security

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/data-security

## Data Fetching Approaches

### 1. External HTTP APIs (for existing large apps)
Use Zero Trust — call existing REST/GraphQL endpoints from Server Components with `fetch`, passing credentials from cookies.

### 2. Data Access Layer (recommended for new projects)
Centralize all data access with auth checks and DTOs:
```ts
// data/auth.ts
import { cache } from 'react'
import { cookies } from 'next/headers'
export const getCurrentUser = cache(async () => {
  const token = cookies().get('AUTH_TOKEN')
  const decoded = await decryptAndValidate(token)
  return new User(decoded.id) // class prevents accidental client passing
})

// data/user-dto.tsx
import 'server-only'
export async function getProfileDTO(slug: string) {
  const [rows] = await sql`SELECT * FROM user WHERE slug = ${slug}`
  const currentUser = await getCurrentUser()
  return {
    username: canSeeUsername(currentUser) ? rows[0].username : null,
    phonenumber: canSeePhoneNumber(currentUser, rows[0].team) ? rows[0].phonenumber : null,
  }
}
```

### 3. Component-level (prototypes only)
Direct DB queries in Server Components — risky since it's easy to pass full objects to Client Components.

## Preventing Data Exposure to Client

### Taint APIs (experimental)
```js
// next.config.js
module.exports = { experimental: { taint: true } }
```
Use `experimental_taintObjectReference` and `experimental_taintUniqueValue` to block sensitive values from reaching the client.

### `server-only` package
```ts
import 'server-only' // Build error if imported on client
```

### Secret environment variables
Only DAL should access `process.env` for secrets. `NEXT_PUBLIC_` prefix exposes vars to browser.

## Server Action Security

### Built-in protections
- Encrypted, non-deterministic action IDs
- Dead code elimination (unused actions removed from client bundle)
- IDs rotated at build time (cached max 14 days)

### Always validate + authorize inside each action
```tsx
// app/actions.ts
'use server'
export async function deletePost(postId: string) {
  const session = await auth()
  if (!session?.user) throw new Error('Unauthorized')
  const post = await db.post.findUnique({ where: { id: postId } })
  if (post.authorId !== session.user.id) throw new Error('Forbidden') // IDOR prevention
  await db.post.delete({ where: { id: postId } })
}
```

### Never trust client input
Always re-verify inside Server Actions, even if the page has an auth check. Page-level auth check does NOT extend to actions.

### Control return values — only return what UI needs
```ts
// BAD: returns full DB record
return db.user.update(...)
// GOOD: returns only what client needs
await db.user.update(...)
return { success: true }
```

### Closures and encryption
Captured variables in closure-based Server Actions are encrypted. For multi-server deployments, set `NEXT_SERVER_ACTIONS_ENCRYPTION_KEY` env var (base64-encoded AES key, 16/24/32 bytes) for consistent decryption across instances.

### CSRF protection
Next.js compares `Origin` header to `Host` header. For reverse proxy setups, configure: `serverActions: { allowedOrigins: ['my-proxy.com'] }`.

### Avoid mutations during rendering
Never delete cookies or revalidate cache inside component render — use Server Actions for mutations.

## Auditing Checklist
- Is there a DAL? Are DB packages/env vars only imported there?
- `"use client"` files: do component props expect private data? Are type signatures overly broad?
- `"use server"` files: are args validated? Is user re-authorized? Does action check resource ownership?
- `/[param]/` folders: are params validated?
- `proxy.ts` and `route.ts`: extra scrutiny + penetration testing