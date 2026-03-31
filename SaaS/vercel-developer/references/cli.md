# Vercel CLI

The Vercel CLI lets you deploy, manage environment variables, configure domains, and debug deployments from your terminal.

## Installation

```bash
npm i -g vercel
```

Verify: `vercel --version`

## Authentication

```bash
# Login (opens browser)
vercel login

# Login with email
vercel login --email

# Logout
vercel logout

# Show current user
vercel whoami
```

## Project Setup

```bash
# Link current directory to an existing Vercel project
vercel link

# Unlink
vercel unlink

# Pull project settings + env vars for local dev
vercel pull

# Pull env vars to .env.local
vercel env pull .env.local

# Pull for a specific environment
vercel env pull --environment=preview
```

## Deployment

```bash
# Deploy preview (default)
vercel deploy

# Deploy to production
vercel deploy --prod
# or shorthand:
vercel --prod

# Force new deployment (bypass cache)
vercel --force

# Build locally using Vercel's build process
vercel build

# Run local dev server with Vercel features (functions, middleware)
vercel dev
```

## Environment Variables

```bash
# Add interactively (prompts for value and environments)
vercel env add <NAME>

# Add from file
echo "value" | vercel env add <NAME> production

# List all env vars
vercel env ls

# Pull to local .env.local
vercel env pull .env.local

# Remove
vercel env rm <NAME> production

# Run a command with Vercel env vars injected
vercel env run -- npm run dev
```

## Domains

```bash
# Add a domain to your project
vercel domains add <domain>

# Inspect domain (shows DNS requirements)
vercel domains inspect <domain>

# List all domains
vercel domains ls

# Remove a domain
vercel domains rm <domain>

# Set a domain alias on a deployment
vercel alias <deployment-url> <domain>

# List aliases
vercel alias ls

# Remove an alias
vercel alias rm <alias>
```

## Inspection & Debugging

```bash
# View deployment details
vercel inspect <deployment-url>

# View deployment logs
vercel logs <deployment-url>

# View error-only logs
vercel logs --deployment <id> --level error

# List all deployments for current project
vercel ls

# Make a request with auto-bypass token (useful for testing)
vercel curl /api/hello

# POST request to deployment
vercel curl /api/users -X POST -d '{"name":"John"}'

# HTTP timing visualization
vercel httpstat /api/hello
```

## Project Management

```bash
# Create a new project
vercel project add <name>

# List all projects
vercel project ls

# Remove a project
vercel project rm <name>

# Remove a specific deployment
vercel rm <deployment-url>

# Switch team/scope
vercel switch

# List teams
vercel teams ls
```

## Rollbacks

```bash
# Roll back production to a previous deployment
vercel rollback <deployment-url>

# Redeploy an existing deployment
vercel redeploy <deployment-url>

# Promote a preview deployment to production
vercel promote <deployment-url>
```

## Storage (Blob, KV)

```bash
# Upload a file to Blob storage
vercel blob put <file>

# Upload with custom path
vercel blob put <file> --pathname assets/hero.jpg

# List blob files
vercel blob list

# Filter by prefix
vercel blob list --prefix images/

# Delete a blob
vercel blob del <url>

# Copy a blob
vercel blob copy <from-url> <to-pathname>

# Create a Blob store
vercel blob store add <name>
```

## Cache Management

```bash
# Invalidate cache by tag
vercel cache invalidate --tag <tag>

# Invalidate multiple tags
vercel cache invalidate --tag blog,users,homepage

# Purge CDN cache
vercel cache purge --type cdn

# Purge data cache
vercel cache purge --type data

# Delete cache by tag (permanent, use with caution)
vercel cache dangerously-delete --tag <tag>
```

## Git Integration

```bash
# Connect Git provider
vercel git connect

# Disconnect
vercel git disconnect
```

## Integrations

```bash
# Install an integration
vercel integration add <name>

# List installed integrations
vercel integration list

# Open integration dashboard
vercel integration open <name>

# Remove an integration
vercel integration remove <name>
```

## Testing & Bisect

```bash
# Binary search to find the deployment that introduced a bug
vercel bisect

# Automated bisect with a test script
vercel bisect --run ./test.sh

# Bisect with known good and bad deployments
vercel bisect --good <url> --bad <url>
```

## Common Workflow: Deploy a SaaS App

```bash
# 1. Clone and enter project
git clone <repo> && cd <repo>

# 2. Install dependencies
npm install

# 3. Link to Vercel project (or create new)
vercel link

# 4. Pull env vars
vercel env pull .env.local

# 5. Develop locally
npm run dev

# 6. Deploy preview
vercel deploy

# 7. Verify preview works, then deploy production
vercel deploy --prod

# 8. Add custom domain
vercel domains add yourdomain.com

# 9. Configure DNS per the output, then verify
vercel domains inspect yourdomain.com
```