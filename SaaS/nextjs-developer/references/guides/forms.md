# Forms

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/forms

Always verify auth inside Server Actions regardless of the page's auth state. See [Data Security](/docs/app/guides/data-security).

## Basic form with Server Action

```tsx
// app/invoices/page.tsx
export default function Page() {
  async function createInvoice(formData: FormData) {
    'use server'
    const session = await auth()
    if (!session?.user) throw new Error('Unauthorized')
    const rawFormData = {
      customerId: formData.get('customerId'),
      amount: formData.get('amount'),
      status: formData.get('status'),
    }
    // mutate + revalidate
  }
  return <form action={createInvoice}>...</form>
}
```

Use `Object.fromEntries(formData)` for many fields (note: result includes `$ACTION_` prefixed properties).

## Passing additional arguments — `bind`

```tsx
// Client Component
const updateUserWithId = updateUser.bind(null, userId)
return <form action={updateUserWithId}><input name="name" /></form>

// Server Action
export async function updateUser(userId: string, formData: FormData) {}
```

Alternative: hidden input `<input type="hidden" name="userId" value={userId} />`.

## Validation errors with `useActionState`

```tsx
// app/ui/signup.tsx
'use client'
import { useActionState } from 'react'
import { createUser } from '@/app/actions'

export function Signup() {
  const [state, formAction, pending] = useActionState(createUser, { message: '' })
  return (
    <form action={formAction}>
      <input type="text" id="email" name="email" required />
      <p aria-live="polite">{state?.message}</p>
      <button disabled={pending}>Sign up</button>
    </form>
  )
}
```

Server Action with `useActionState` receives `initialState` as first arg:
```ts
export async function createUser(initialState: any, formData: FormData) {
  const validatedFields = schema.safeParse({ email: formData.get('email') })
  if (!validatedFields.success) return { errors: validatedFields.error.flatten().fieldErrors }
}
```

## `useFormStatus` for pending state (separate component)

```tsx
'use client'
import { useFormStatus } from 'react-dom'
export function SubmitButton() {
  const { pending } = useFormStatus()
  return <button disabled={pending} type="submit">Sign Up</button>
}
```

> In React 19, `useFormStatus` also exposes `data`, `method`, `action`.

## Optimistic updates

```tsx
'use client'
import { useOptimistic } from 'react'
export function Thread({ messages }) {
  const [optimisticMessages, addOptimisticMessage] = useOptimistic(
    messages,
    (state, newMessage) => [...state, { message: newMessage }]
  )
  const formAction = async (formData: FormData) => {
    const message = formData.get('message') as string
    addOptimisticMessage(message)
    await send(message)
  }
  return (
    <div>
      {optimisticMessages.map((m, i) => <div key={i}>{m.message}</div>)}
      <form action={formAction}>
        <input type="text" name="message" /><button type="submit">Send</button>
      </form>
    </div>
  )
}
```

## Programmatic submission

```tsx
const handleKeyDown = (e: React.KeyboardEvent<HTMLTextAreaElement>) => {
  if ((e.ctrlKey || e.metaKey) && (e.key === 'Enter' || e.key === 'NumpadEnter')) {
    e.preventDefault()
    e.currentTarget.form?.requestSubmit()
  }
}
```

## Nested form elements

`<button>`, `<input type="submit">`, `<input type="image">` inside a form accept the `formAction` prop to call different Server Actions from the same form.