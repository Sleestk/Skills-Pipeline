# Environment Variables

Environment variables on Vercel control secrets, API keys, and configuration. They are baked into the build — changing them requires a redeploy.

## The Critical Rule: `NEXT_PUBLIC_` Prefix

| Prefix               | Accessible Where    | Use For                              |
|----------------------|---------------------|--------------------------------------|
| `NEXT_PUBLIC_`       | Client + Server     | Publishable keys, public API URLs    |
| No prefix            | Server only         | Secret keys, database URLs, tokens   |

**Never put `NEXT_PUBLIC_` on secret keys.** They become visible in the browser's JavaScript bundle.

```env
# ✅ Correct
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_...   # Safe for client
STRIPE_SECRET_KEY=sk_test_...                     # Server only

# ❌ Wrong — exposes secret to browser
NEXT_PUBLIC_STRIPE_SECRET_KEY=sk_test_...
```

## Three Environments

Vercel supports three environments with independent env var values:

| Environment   | When Used                        | Description                          |
|---------------|----------------------------------|--------------------------------------|
| Development   | `vercel dev` / `vercel env pull` | Local development                    |
| Preview       | Non-production branch deploys    | Staging, QA, PR previews             |
| Production    | `main` branch deploys            | Live site                            |

You can set different values per environment. Example: test Stripe keys for Preview, live keys for Production.

## Adding Environment Variables

### Via Dashboard (Recommended)
1. Go to **Project Settings → Environment Variables**
2. Enter the key name and value
3. Select which environments it applies to (Development, Preview, Production)
4. Click **Save**
5. **Redeploy** for changes to take effect

### Via CLI

```bash
# Add interactively
vercel env add STRIPE_SECRET_KEY

# Add with a specific value (from file)
echo "sk_test_xxx" | vercel env add STRIPE_SECRET_KEY production

# List all env vars
vercel env ls

# Pull env vars to local .env.local
vercel env pull .env.local

# Pull for a specific environment
vercel env pull --environment=preview

# Remove an env var
vercel env rm STRIPE_SECRET_KEY production
```

## Common SaaS Stack Variables

```env
# Supabase
NEXT_PUBLIC_SUPABASE_URL=https://xxxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbGci...
SUPABASE_SERVICE_ROLE_KEY=eyJhbGci...

# Stripe
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_...
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...

# App
NEXT_PUBLIC_APP_URL=https://yourdomain.com
```

## System Environment Variables

Vercel automatically provides these (no configuration needed):

| Variable               | Value                                    |
|------------------------|------------------------------------------|
| `VERCEL`               | `1` (always set on Vercel)               |
| `VERCEL_ENV`           | `production`, `preview`, or `development`|
| `VERCEL_URL`           | Deployment URL (without protocol)        |
| `VERCEL_BRANCH_URL`    | Branch-specific URL                      |
| `VERCEL_PROJECT_PRODUCTION_URL` | Production domain               |
| `VERCEL_GIT_COMMIT_SHA`| Git commit SHA                           |
| `VERCEL_GIT_COMMIT_MESSAGE` | Git commit message                  |
| `VERCEL_GIT_REPO_SLUG` | Repository name                         |
| `VERCEL_GIT_REPO_OWNER`| Repository owner                        |

Use `VERCEL_URL` to construct URLs dynamically:
```ts
const baseUrl = process.env.VERCEL_URL
  ? `https://${process.env.VERCEL_URL}`
  : 'http://localhost:3000'
```

## Sensitive Variables (Encrypted)

Vercel encrypts all environment variables at rest. For extra-sensitive values, use **Sensitive Environment Variables** in the Dashboard — these are masked and cannot be read back after creation.

## Common Mistakes

1. **Forgetting to redeploy after adding env vars.** Env vars are baked at build time. New vars require a new deployment.
2. **Using `NEXT_PUBLIC_` on secrets.** This exposes them in client-side JavaScript.
3. **Not setting vars for the right environment.** Preview deploys use Preview env vars. Production deploys use Production env vars. If your Preview deploy is broken, check that Preview env vars are set.
4. **Using `process.env` in client components without `NEXT_PUBLIC_`.** The value will be `undefined` at runtime.
5. **Hardcoding URLs that change per environment.** Use `VERCEL_URL` or set `NEXT_PUBLIC_APP_URL` per environment.