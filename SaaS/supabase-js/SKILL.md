---
name: supabase-js
description: Full-stack Supabase developer skill for building Next.js applications with Supabase. Covers Database (Postgres tables, schemas, views, indexes, joins, JSON, RLS, CLS, roles, functions, triggers, webhooks, event triggers), Auth (email/password, magic link, OTP, OAuth, anonymous sign-ins, MFA, SSR/PKCE flows, sessions, redirect URLs, user management), the JavaScript client (`@supabase/supabase-js` — select, insert, update, upsert, delete, rpc, filters, modifiers, auth methods), and Next.js integration (`@supabase/ssr`, server/browser clients, proxy/middleware, cookie-based auth). Use this skill whenever the user mentions Supabase, supabase-js, @supabase/ssr, createClient from Supabase, RLS policies, Row Level Security with Supabase, Supabase Auth, supabase.auth, supabase.from, Postgres with Supabase, Supabase database, Supabase Next.js, or any task involving building with Supabase. Even if the user just says "set up auth" or "create a table" in the context of a Supabase project — trigger this skill immediately.
---

# Supabase Developer Skill

A comprehensive skill for building Next.js applications with Supabase. Covers four domains: **Database**, **Auth**, **JavaScript Client**, and **Next.js Integration**.

## Architecture

This skill uses progressive disclosure with nested routing:

```
supabase/
├── SKILL.md                          ← You are here (orchestrator)
├── references/
│   ├── database/
│   │   ├── README.md                 ← Database router
│   │   ├── fundamentals.md           ← Connecting, importing, securing
│   │   ├── basics.md                 ← Tables, views, arrays, indexes, joins, JSON
│   │   ├── intermediate.md           ← Cascade deletes, enums, functions, triggers, webhooks, event triggers
│   │   └── access-security.md        ← RLS, CLS, Postgres roles, custom roles
│   ├── auth.md                       ← Full auth reference
│   ├── js-client.md                  ← @supabase/supabase-js API reference
│   └── nextjs.md                     ← @supabase/ssr + Next.js integration
```

## Routing

When the user's request arrives, determine which domain(s) it touches, then read the appropriate reference file(s):

### Database
If the request involves tables, schemas, views, columns, data types, indexes, joins, foreign keys, JSON/JSONB, importing data, connecting to the database, or any SQL DDL/DML:
→ Read `references/database/README.md` first. It will route you to the correct sub-file.

### Auth
If the request involves sign up, sign in, sign out, password reset, OAuth, magic link, OTP, MFA, sessions, JWTs, RLS policies tied to `auth.uid()`, redirect URLs, user management, or the `auth` schema:
→ Read `references/auth.md`

### JavaScript Client
If the request involves `supabase.from()`, `.select()`, `.insert()`, `.update()`, `.delete()`, `.rpc()`, filters (`.eq()`, `.neq()`, `.in()`, etc.), modifiers (`.order()`, `.limit()`, `.single()`), or any `supabase.auth.*` method calls:
→ Read `references/js-client.md`

### Next.js Integration
If the request involves `@supabase/ssr`, `createBrowserClient`, `createServerClient`, middleware/proxy auth, cookie-based sessions, server components with Supabase, or wiring Supabase into a Next.js app:
→ Read `references/nextjs.md`

### Multiple Domains
Many requests touch multiple domains. For example, "set up auth with email/password in my Next.js app" requires **Auth** + **Next.js** + **JS Client**. Read all relevant files.

## Default Stack

- **Framework**: Next.js (App Router)
- **Client library**: `@supabase/supabase-js` v2
- **SSR package**: `@supabase/ssr`
- **Language**: TypeScript
- **ORM**: None (direct supabase-js queries)

## Key Principles

1. **Always enable RLS** on public-facing tables. No exceptions.
2. **Use `(select auth.uid())` in RLS policies** (wrapped in select for performance).
3. **Never expose the service_role key** to the client. Use the anon/publishable key.
4. **Use PKCE flow** for SSR authentication (default in `@supabase/ssr`).
5. **Create a new Supabase client per request** on the server. The browser client is a singleton.
6. **Use `supabase.auth.getClaims()`** in server code to validate JWTs. Never trust `getSession()` on the server.
7. **Add indexes** on columns used in RLS policies.
8. **Specify the `TO` role** in RLS policies (`to authenticated`, `to anon`).