# Deploying

> Next.js 16.2.1 — https://nextjs.org/docs/app/getting-started/deploying

| Option | Feature Support |
|---|---|
| Node.js server | All |
| Docker container | All |
| Static export | Limited |
| Adapters | Varies (verified adapters run the full test suite) |

---

## Node.js Server

Any provider that supports Node.js. Ensure `package.json` has build + start scripts:

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start"
  }
}
```

Run `npm run build` → `npm run start`. Supports all Next.js features. Can eject to a [custom server](https://nextjs.org/docs/app/guides/custom-server) if needed. See [self-hosting guide](https://nextjs.org/docs/app/guides/self-hosting) for infrastructure configuration.

**Templates:** Flightcontrol, Railway, Replit, Hostinger

---

## Docker

Any provider supporting Docker containers (Kubernetes, cloud providers). Supports all Next.js features.

> Use local dev (`npm run dev`) instead of Docker during development on Mac/Windows for better performance.

**Templates:**
- [`with-docker`](https://github.com/vercel/next.js/tree/canary/examples/with-docker) — `output: "standalone"` for minimal production image
- [`with-docker-export-output`](https://github.com/vercel/next.js/tree/canary/examples/with-docker-export-output) — `output: "export"` for fully static container
- [`with-docker-multi-env`](https://github.com/vercel/next.js/tree/canary/examples/with-docker-multi-env) — separate configs for dev/staging/prod

**Hosting provider guides:** DigitalOcean, Fly.io, Google Cloud Run, Render, SST

---

## Static Export

Deploy as a static site or SPA with no server required.

```js
// next.config.js
module.exports = { output: 'export' }
```

Can be hosted on any static asset server: AWS S3, Nginx, Apache, GitHub Pages.

**Limitation:** Does not support features that require a server. See [static exports docs](https://nextjs.org/docs/app/guides/static-exports#unsupported-features) for what's unsupported.

**Template:** GitHub Pages

---

## Adapters

The [Deployment Adapter API](https://nextjs.org/docs/app/api-reference/config/next-config-js/adapterPath) lets platforms customize how Next.js apps are built and deployed.

### Verified Adapters
Open source, run the full Next.js compatibility test suite, hosted under the [Next.js GitHub org](https://github.com/nextjs):
- **Vercel** — https://vercel.com/docs/frameworks/nextjs
- **Bun** — https://bun.sh/docs/frameworks/nextjs

Cloudflare and Netlify are actively building verified adapters.

### Other Platforms (own integrations, not verified)
- [Appwrite Sites](https://appwrite.io/docs/products/sites/quick-start/nextjs)
- [AWS Amplify Hosting](https://docs.amplify.aws/nextjs/start/quickstart/nextjs-app-router-client-components)
- [Cloudflare Workers](https://developers.cloudflare.com/workers/frameworks/framework-guides/nextjs)
- [Deno Deploy](https://docs.deno.com/examples/next_tutorial)
- [Firebase App Hosting](https://firebase.google.com/docs/app-hosting/get-started)
- [Netlify](https://docs.netlify.com/frameworks/next-js/overview/)

Feature support and compatibility vary — check each provider's docs.