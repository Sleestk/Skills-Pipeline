# Auth Reference

## Overview

Supabase Auth provides authentication (verifying identity) and authorization (controlling access). Uses JWTs. Integrates with RLS for row-level authorization.

### Architecture
1. **Client layer** — supabase-js handles HTTP calls, token persistence, refresh
2. **Kong API gateway** — shared across Supabase products
3. **Auth service** (GoTrue) — validates/issues JWTs, communicates with providers
4. **Postgres** — stores user data in `auth` schema (not exposed via API)

## Users

- **Permanent users**: tied to email, phone, or OAuth identity
- **Anonymous users**: have a user_id but no identity to sign back in
- Both use the `authenticated` Postgres role
- `anon` role = no user at all (unauthenticated)

### Key user object fields
`id`, `email`, `phone`, `email_confirmed_at`, `app_metadata` (provider info, not user-editable), `user_metadata` (user-editable — don't use for authorization), `identities`, `is_anonymous`, `created_at`

### Retrieve user
```js
const { data: { user } } = await supabase.auth.getUser()
```

## Identities

A user can have multiple identities (email, phone, OAuth, SAML). Key fields: `provider`, `provider_id`, `identity_data`, `email`.

## Sessions

- Access token (JWT): short-lived (default 1 hour)
- Refresh token: unique string, single-use, exchanged for new token pair
- Session terminates on: sign out, password change, inactivity timeout, max lifetime, sign in on another device

### Session flows
- **Implicit flow**: tokens in URL fragment (`#access_token=...`), client-only
- **PKCE flow**: auth code in URL query (`?code=...`), exchange server-side — **use this for SSR**

### Session configuration (Pro+ plans)
- **Time-box**: terminate after fixed duration
- **Inactivity timeout**: terminate if not refreshed
- **Single session per user**: only most recent session active

## Sign Up

### Email + password
```js
const { data, error } = await supabase.auth.signUp({
  email: 'user@example.com',
  password: 'secure-password',
  options: { emailRedirectTo: 'https://example.com/welcome' },
})
```

### With metadata
```js
const { data, error } = await supabase.auth.signUp({
  email: 'user@example.com',
  password: 'secure-password',
  options: {
    data: { first_name: 'John', age: 27 },
  },
})
```

## Sign In

### Email + password
```js
const { data, error } = await supabase.auth.signInWithPassword({
  email: 'user@example.com',
  password: 'secure-password',
})
```

### Magic Link
```js
const { data, error } = await supabase.auth.signInWithOtp({
  email: 'user@example.com',
  options: {
    shouldCreateUser: false,
    emailRedirectTo: 'https://example.com/welcome',
  },
})
```

### Email OTP (6-digit code)
```js
// Step 1: Send OTP
const { data, error } = await supabase.auth.signInWithOtp({
  email: 'user@example.com',
  options: { shouldCreateUser: false },
})

// Step 2: Verify OTP
const { data: { session }, error } = await supabase.auth.verifyOtp({
  email: 'user@example.com',
  token: '123456',
  type: 'email',
})
```

To send OTP instead of Magic Link, modify the email template to include `{{ .Token }}`.

### OAuth (social login)
```js
const { data, error } = await supabase.auth.signInWithOAuth({
  provider: 'google',
  options: { redirectTo: 'https://example.com/auth/callback' },
})
```

### Anonymous
```js
const { data, error } = await supabase.auth.signInAnonymously()
```
Anonymous users use `authenticated` role. Check `is_anonymous` in JWT for RLS:
```sql
create policy "Only permanent users can post"
on news_feed as restrictive for insert to authenticated
with check ((select (auth.jwt()->>'is_anonymous')::boolean) is false);
```

## Sign Out
```js
await supabase.auth.signOut()           // global (all sessions)
await supabase.auth.signOut({ scope: 'local' })  // current session only
await supabase.auth.signOut({ scope: 'others' }) // all except current
```

## Password Reset

### Step 1: Request reset email
```js
await supabase.auth.resetPasswordForEmail('user@example.com', {
  redirectTo: 'http://example.com/account/update-password',
})
```

### Step 2: Update password (after user clicks link)
```js
await supabase.auth.updateUser({ password: 'new_password' })
```

For PKCE flow, update email template to use `{{ .TokenHash }}` and create a `/auth/confirm` route to exchange the token.

## Auth Events
```js
supabase.auth.onAuthStateChange((event, session) => {
  console.log(event, session)
})
// Events: SIGNED_IN, SIGNED_OUT, TOKEN_REFRESHED, USER_UPDATED, PASSWORD_RECOVERY
```

## Multi-Factor Authentication (MFA)

### Assurance levels
- `aal1`: conventional login (password, OTP, OAuth)
- `aal2`: verified with additional factor (TOTP or phone)

### Enrollment (TOTP)
```js
// 1. Enroll
const { data, error } = await supabase.auth.mfa.enroll({ factorType: 'totp' })
// data.totp.qr_code contains SVG QR code

// 2. Challenge
const { data: challenge } = await supabase.auth.mfa.challenge({ factorId: data.id })

// 3. Verify
const { data: verify } = await supabase.auth.mfa.verify({
  factorId: data.id,
  challengeId: challenge.id,
  code: userCode,
})
```

### Check AAL level
```js
const { data } = await supabase.auth.mfa.getAuthenticatorAssuranceLevel()
// data.currentLevel: 'aal1' | 'aal2'
// data.nextLevel: 'aal1' | 'aal2'
// If currentLevel='aal1' && nextLevel='aal2' → show MFA challenge
```

### RLS enforcement
```sql
-- Enforce for all users
create policy "Require MFA" on table_name as restrictive to authenticated
using ((select auth.jwt()->>'aal') = 'aal2');

-- Enforce only for users who enrolled
create policy "Require MFA if enrolled" on table_name as restrictive to authenticated
using (
  array[(select auth.jwt()->>'aal')] <@ (
    select case when count(id) > 0 then array['aal2'] else array['aal1', 'aal2'] end
    from auth.mfa_factors where (select auth.uid()) = user_id and status = 'verified'
  )
);
```

## Redirect URLs

Configure in dashboard under URL Configuration. Support wildcards:
- `http://localhost:3000/**` — local dev
- `https://*-<team>.vercel.app/**` — Vercel previews
- `https://**--my_org.netlify.app/**` — Netlify previews

### Dynamic redirect for Vercel
```js
const getURL = () => {
  let url = process?.env?.NEXT_PUBLIC_SITE_URL ?? process?.env?.NEXT_PUBLIC_VERCEL_URL ?? 'http://localhost:3000/'
  url = url.startsWith('http') ? url : `https://${url}`
  url = url.endsWith('/') ? url : `${url}/`
  return url
}
```

## User Management

### Public profiles pattern
```sql
create table public.profiles (
  id uuid not null references auth.users on delete cascade,
  first_name text,
  last_name text,
  primary key (id)
);
alter table public.profiles enable row level security;
```

### Auto-create profile on signup
```sql
create function public.handle_new_user()
returns trigger language plpgsql security definer set search_path = '' as $$
begin
  insert into public.profiles (id, first_name, last_name)
  values (new.id, new.raw_user_meta_data ->> 'first_name', new.raw_user_meta_data ->> 'last_name');
  return new;
end;
$$;

create trigger on_auth_user_created
  after insert on auth.users
  for each row execute procedure public.handle_new_user();
```

### Deleting users
You cannot delete a user who owns Storage objects. Delete their objects first.

### Exporting users
```sql
select * from auth.users;
```

## General Configuration
- **Allow new users to sign up**: toggle in Auth settings
- **Confirm Email**: enabled by default on hosted projects
- **Allow anonymous sign-ins**: toggle in Auth settings
- **Allow manual linking**: required for converting anonymous → permanent users