---
name: vercel-developer
description: >
  Full-stack Vercel developer skill for deploying and managing Next.js applications. Covers Deployment (preview, production, Git integration, CLI), Environment Variables (per-environment config, NEXT_PUBLIC_ prefix, secrets), Domains (custom domains, DNS, SSL), Vercel CLI (deploy, env, link, pull, dev, logs), Serverless & Edge Functions, Storage (Blob, KV, Postgres), Caching (ISR, CDN, revalidation), and Integrations (Supabase, Stripe). Use this skill whenever the user mentions Vercel, deploying, vercel deploy, vercel CLI, preview deployment, production deployment, vercel env, vercel domains, serverless functions, edge functions, custom domains, or any task involving deploying a Next.js app. Even if the user just says "deploy this", "push to production", "go live", or "set up hosting" — trigger immediately. Also trigger for deploying a Stripe + Supabase + Next.js stack to Vercel.
---

# Vercel Developer Skill

A comprehensive skill for deploying and managing Next.js applications on Vercel. Covers five domains: **Deployment**, **Environment Variables**, **Domains & SSL**, **CLI**, and **Platform Features**.

## Architecture

```
vercel-developer/
├── SKILL.md                          ← You are here (orchestrator)
├── references/
│   ├── deployment.md                 ← Deploy flows (Git, CLI, preview, production)
│   ├── environment-variables.md      ← Env var management, secrets, per-environment config
│   ├── domains-ssl.md                ← Custom domains, DNS, SSL certificates
│   ├── cli.md                        ← Full Vercel CLI command reference
│   └── platform-features.md          ← Storage, caching, functions, integrations
```

## Routing

### Deployment
If the request involves deploying an app, Git integration, preview deployments, production deploys, rollbacks, CI/CD, or the deployment pipeline:
→ Read `references/deployment.md`

### Environment Variables
If the request involves setting env vars, the `NEXT_PUBLIC_` prefix, secrets management, per-environment configuration (development/preview/production), or pulling env vars locally:
→ Read `references/environment-variables.md`

### Domains & SSL
If the request involves custom domains, DNS configuration, SSL certificates, domain verification, or domain aliases:
→ Read `references/domains-ssl.md`

### CLI
If the request involves Vercel CLI commands (`vercel`, `vercel deploy`, `vercel env`, `vercel link`, `vercel pull`, `vercel dev`, `vercel logs`, etc.):
→ Read `references/cli.md`

### Platform Features
If the request involves Vercel-specific features like Serverless Functions, Edge Functions, Vercel Blob/KV/Postgres storage, caching (ISR, CDN), Web Application Firewall, Speed Insights, Analytics, or integrations:
→ Read `references/platform-features.md`

### Multiple Domains
Many requests touch multiple domains. For example, "deploy my Next.js + Stripe + Supabase app with a custom domain" requires **Deployment** + **Environment Variables** + **Domains & SSL**. Read all relevant files.

## Default Stack

- **Framework**: Next.js (App Router)
- **Deployment target**: Vercel
- **CLI**: `vercel` (latest)
- **Language**: TypeScript

## Key Principles

1. **Zero-config for Next.js.** Vercel auto-detects Next.js and configures builds, serverless functions, and routing automatically.
2. **Git push = deploy.** Connect a GitHub/GitLab/Bitbucket repo. Every push to `main` triggers a production deployment. Every PR gets a preview deployment.
3. **Environment variables must be set before deployment.** Env vars are baked into the build. If you add one after deploying, you must redeploy.
4. **`NEXT_PUBLIC_` prefix exposes to the browser.** Without this prefix, env vars are server-only. Never prefix secret keys with `NEXT_PUBLIC_`.
5. **Three environments:** Development, Preview, Production. Each can have different env var values.
6. **Preview deployments are free and automatic.** Every push to a non-production branch creates a preview with a unique URL.
7. **Rollbacks are instant.** You can roll back to any previous deployment via Dashboard or CLI.
8. **Edge Functions run globally.** Serverless Functions run in a single region by default.

## Quick Start

```bash
# Install CLI
npm i -g vercel

# Login
vercel login

# Link your project
vercel link

# Pull env vars for local dev
vercel env pull .env.local

# Deploy preview
vercel deploy

# Deploy to production
vercel --prod
```

## Environment Variables for a SaaS Stack

When deploying a Next.js + Supabase + Stripe app, set these in Vercel:

| Variable                                | Prefix?  | Description                          |
|-----------------------------------------|----------|--------------------------------------|
| `NEXT_PUBLIC_SUPABASE_URL`              | Yes      | Supabase project URL (client-safe)   |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY`         | Yes      | Supabase anon key (client-safe)      |
| `SUPABASE_SERVICE_ROLE_KEY`             | No       | Supabase service role (server-only)  |
| `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY`    | Yes      | Stripe publishable key (client-safe) |
| `STRIPE_SECRET_KEY`                     | No       | Stripe secret key (server-only)      |
| `STRIPE_WEBHOOK_SECRET`                 | No       | Stripe webhook signing secret        |