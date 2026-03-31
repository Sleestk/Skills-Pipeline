# Multi-Tenant Apps

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/multi-tenant

For building a single Next.js application that serves multiple tenants, use the [Platforms Starter Kit](https://vercel.com/templates/next.js/platforms-starter-kit) as a reference architecture.

The starter kit demonstrates:
- Subdomain-based routing via `proxy.ts` (e.g. `tenant.yourapp.com`)
- Wildcard domain support
- Per-tenant data isolation
- Custom domain support

Source: https://github.com/vercel/platforms