# Deployment

Vercel deployment is zero-config for Next.js. Connect a Git repo and every push deploys automatically.

## Three Deployment Types

| Type        | Trigger                          | URL                                    | Environment     |
|-------------|----------------------------------|----------------------------------------|-----------------|
| Production  | Push to `main` (or prod branch)  | `yourdomain.com`                       | Production      |
| Preview     | Push to any other branch / PR    | `project-git-branch-team.vercel.app`   | Preview         |
| Instant Rollback | Manual via Dashboard / CLI | Reverts production to a previous deploy | Production     |

## Deploy via Git (Recommended)

### Setup
1. Go to [vercel.com/new](https://vercel.com/new)
2. Import your GitHub/GitLab/Bitbucket repo
3. Vercel auto-detects Next.js and configures the build
4. Add environment variables
5. Click **Deploy**

### After Setup
- **Every push to `main`** → production deployment
- **Every PR or branch push** → preview deployment with unique URL
- **PR comments** show preview URL automatically

### Build Settings (auto-detected for Next.js)
| Setting           | Default Value        |
|-------------------|----------------------|
| Framework Preset  | Next.js              |
| Build Command     | `next build`         |
| Output Directory  | `.next`              |
| Install Command   | `npm install`        |
| Node.js Version   | 20.x (configurable)  |

## Deploy via CLI

```bash
# Preview deployment (default)
vercel deploy

# Production deployment
vercel deploy --prod
# or
vercel --prod

# Force new deployment (skip cache)
vercel --force

# Deploy specific directory
vercel deploy ./my-app
```

## Deploy Workflow: CLI from Scratch

```bash
# 1. Link local project to Vercel
vercel link

# 2. Pull env vars for local development
vercel env pull .env.local

# 3. Develop locally
npm run dev

# 4. Deploy a preview
vercel deploy

# 5. Verify the preview
# (visit the URL printed by the CLI)

# 6. Deploy to production
vercel deploy --prod

# 7. Add a custom domain (if needed)
vercel domains add yourdomain.com
```

## Preview Deployments

Every non-production push gets a unique preview URL. Previews:

- Use **Preview** environment variables
- Are accessible via `https://project-git-branch-team.vercel.app`
- Show in PR comments on GitHub/GitLab
- Are free and unlimited
- Useful for QA, design review, stakeholder demos

## Rollbacks

```bash
# Roll back to a specific deployment
vercel rollback <deployment-url>

# Or from the Dashboard:
# Deployments → select a previous deploy → Promote to Production
```

Rollbacks are instant — Vercel re-routes traffic to the previous build without rebuilding.

## Build Output

Vercel uses the Next.js build output to create:

- **Static pages** → served from CDN edge
- **Server-rendered pages** → served via Serverless Functions
- **API routes / Route Handlers** → served via Serverless Functions
- **Middleware / Proxy** → served via Edge Functions (global)

## Monorepo Support

For monorepos (Turborepo, Nx, etc.):

```bash
# In Vercel project settings, set:
# Root Directory: apps/web
# or configure in vercel.json:
```

```json
// vercel.json (at repo root)
{
  "projects": [
    { "src": "apps/web", "use": "@vercel/next" }
  ]
}
```

## Ignoring Builds

To skip builds on certain pushes, add to project settings or `vercel.json`:

```json
{
  "git": {
    "deploymentEnabled": {
      "main": true,
      "dev": true,
      "feature/*": false
    }
  }
}
```

Or use the **Ignored Build Step** in project settings with a custom script.

## Deployment Logs

```bash
# View logs for a deployment
vercel logs <deployment-url>

# View error-only logs
vercel logs --deployment <id> --level error

# Inspect deployment details
vercel inspect <deployment-url>
```

From the Dashboard: **Deployments → select deployment → View Function Logs / Build Logs**