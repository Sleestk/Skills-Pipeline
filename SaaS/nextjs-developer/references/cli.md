# Next.js CLI Reference (v16.2.1)

Next.js ships two CLI tools:
- **`create-next-app`** — scaffold a new project
- **`next`** — dev, build, start, and more

---

## `create-next-app`

```bash
pnpm create next-app [project-name] [options]
npx create-next-app@latest [project-name] [options]
yarn create next-app [project-name] [options]
bun create next-app [project-name] [options]
```

### Options

| Option | Description |
|---|---|
| `--ts` / `--typescript` | TypeScript (default) |
| `--js` / `--javascript` | JavaScript |
| `--tailwind` | Tailwind CSS (default) |
| `--react-compiler` | React Compiler enabled |
| `--eslint` | ESLint config |
| `--biome` | Biome (linter + formatter) |
| `--no-linter` | Skip linter |
| `--app` | App Router |
| `--api` | Route handlers only |
| `--src-dir` | Use `src/` directory |
| `--turbopack` | Force Turbopack (default) |
| `--webpack` | Force Webpack |
| `--import-alias <alias>` | Import alias (default `@/*`) |
| `--empty` | Empty project |
| `--use-npm/pnpm/yarn/bun` | Force package manager |
| `-e` / `--example [name\|url]` | Bootstrap from example |
| `--example-path <path>` | Path within example repo |
| `--agents-md` | Include `AGENTS.md` + `CLAUDE.md` (default) |
| `--yes` | Use previous/default preferences |
| `--skip-install` | Skip package install |
| `--disable-git` | Skip git init |
| `--reset-preferences` | Clear stored preferences |

### Interactive prompts (default template)

```
What is your project named? my-app
Would you like to use the recommended Next.js defaults?
  → Yes (TypeScript, ESLint, Tailwind, App Router, AGENTS.md)
  → No, reuse previous settings
  → No, customize settings
```

Customize settings prompts: TypeScript, linter (ESLint/Biome/None), React Compiler, Tailwind, `src/`, App Router, import alias, AGENTS.md.

### Linter options
- **ESLint** — traditional, includes `@next/eslint-plugin-next` rules
- **Biome** — fast modern linter+formatter, built-in Next.js/React support
- **None** — skip, add later

### Bootstrap from example

```bash
# Official Next.js example
npx create-next-app@latest --example [example-name] [project-name]

# Any public GitHub repo
npx create-next-app@latest --example "https://github.com/.../" [project-name]
```

All official examples: https://github.com/vercel/next.js/tree/canary/examples

---

## `next` CLI

```bash
pnpm next [command] [options]
npx next [command] [options]   # use -- before flags with npm
yarn next [command] [options]
bunx next [command] [options]
```

Running `next` without a command is an alias for `next dev`.

### Commands

| Command | Description |
|---|---|
| `dev` | Dev server with HMR, error reporting |
| `build` | Optimized production build |
| `start` | Production server (requires prior `next build`) |
| `info` | System info for bug reports |
| `telemetry` | Enable/disable anonymous telemetry |
| `typegen` | Generate TS route types without a full build |
| `upgrade` | Upgrade to latest Next.js version |
| `experimental-analyze` | Analyze bundle with Turbopack (no build artifacts) |

---

### `next dev`

> Dev builds output to `.next/dev` — safe to run concurrently with `next build`.

| Option | Description |
|---|---|
| `--turbopack` / `--turbo` | Force Turbopack (default) |
| `--webpack` | Use Webpack instead |
| `-p` / `--port <port>` | Port (default: 3000, env: `PORT`) |
| `-H` / `--hostname <hostname>` | Hostname (default: 0.0.0.0) |
| `--experimental-https` | Self-signed HTTPS cert |
| `--experimental-https-key <path>` | Custom HTTPS key |
| `--experimental-https-cert <path>` | Custom HTTPS cert |
| `--experimental-https-ca <path>` | Custom CA cert |
| `--experimental-cpu-prof` | CPU profiling (saves to `.next/cpu-profiles/`) |
| `--experimental-upload-trace <url>` | Upload debug trace to remote HTTP URL |

---

### `next build`

Output legend:
```
○  (Static)   prerendered as static content
ƒ  (Dynamic)  server-rendered on demand
```

| Option | Description |
|---|---|
| `--turbopack` / `--turbo` | Force Turbopack (default) |
| `--webpack` | Use Webpack |
| `-d` / `--debug` | Verbose output (rewrites, redirects, headers) |
| `--profile` | React production profiling |
| `--no-lint` | Disable linting (note: lint removed from build in v16) |
| `--no-mangling` | Disable name mangling (debug only) |
| `--experimental-app-only` | Build only App Router routes |
| `--debug-prerender` | Debug prerender errors (dev only, see below) |
| `--debug-build-paths=<patterns>` | Build specific routes only (comma-sep, glob ok) |
| `--experimental-cpu-prof` | CPU profiling |

**`--debug-prerender` effects:** disables server minification, generates server source maps, continues past first error. **Do not deploy debug builds.**

**`--debug-build-paths` examples:**
```bash
next build --debug-build-paths="app/page.tsx"
next build --debug-build-paths="app/page.tsx,pages/index.tsx"
next build --debug-build-paths="app/**/page.tsx"
```

---

### `next start`

| Option | Description |
|---|---|
| `-p` / `--port <port>` | Port (default: 3000) |
| `-H` / `--hostname <hostname>` | Hostname (default: 0.0.0.0) |
| `--keepAliveTimeout <ms>` | Max ms before closing inactive connections |
| `--experimental-cpu-prof` | CPU profiling |

**keepAliveTimeout tip:** when behind a load balancer (e.g. AWS ELB/ALB), set this *higher* than the proxy's timeout to prevent premature TCP connection termination:
```bash
next start --keepAliveTimeout 70000
```

---

### `next info`

Prints system info (OS, Node, npm/yarn/pnpm, package versions, Next.js config) for bug reports.

```bash
next info --verbose  # additional debug info
```

---

### `next telemetry`

Completely anonymous. Opt-out anytime.

```bash
next telemetry --enable
next telemetry --disable
```

---

### `next typegen`

Generates TypeScript route type definitions without a full build. Useful for CI type-checking.

```bash
next typegen            # current directory
next typegen ./apps/web # specific app
```

Output: `.next/dev/types` (dev) or `.next/types` (prod). Also generates `next-env.d.ts` — add to `.gitignore`.

```bash
# CI pattern
next typegen && tsc --noEmit
```

> Loads `next.config.*` using production build phase — ensure env vars are available.

---

### `next upgrade`

```bash
next upgrade                        # latest
next upgrade --revision 15.0.0      # specific version
next upgrade --revision canary
next upgrade --verbose
```

---

### `next experimental-analyze`

Analyzes bundle composition using Turbopack. No build artifacts produced.

```bash
npx next experimental-analyze               # opens browser UI
npx next experimental-analyze --output      # write to .next/diagnostics/analyze
npx next experimental-analyze --port 4000   # custom port (default: 4000)
```

Browser UI features: filter by route, client/server toggle, full import chain, server-to-client boundary tracing.

---

## Common examples

### Change port
```bash
next dev -p 4000
PORT=4000 next dev   # PORT cannot be set in .env
```

### HTTPS in dev
```bash
next dev --experimental-https
next dev --experimental-https --experimental-https-key ./certs/key.pem --experimental-https-cert ./certs/cert.pem
```

### Pass Node.js args
```bash
NODE_OPTIONS='--inspect' next
NODE_OPTIONS='--throw-deprecation' next build
```

### CPU profiling
```bash
next build --experimental-cpu-prof   # saves to .next/cpu-profiles/
next dev --experimental-cpu-prof     # profile saved on Ctrl+C
next start --experimental-cpu-prof
```

Profile naming:
- `next dev`: `dev-main-*` (orchestration), `dev-server-*` (request handling — analyze this one)
- `next build` (Turbopack): `build-main-*`, `build-turbopack-*`
- `next build` (Webpack): `build-main-*`, `build-webpack-client-*`, `build-webpack-server-*`, `build-webpack-edge-server-*`
- `next start`: `start-main-*`

Open `.cpuprofile` files in Chrome DevTools → Performance tab → Load profile.

---

## Version history

| Version | Change |
|---|---|
| v16.1.0 | Added `next upgrade`, `next experimental-analyze` |
| v16.0.0 | JS bundle size metrics removed from `next build` |
| v15.5.0 | Added `next typegen` |
| v15.4.0 | Added `--debug-prerender` for `next build` |