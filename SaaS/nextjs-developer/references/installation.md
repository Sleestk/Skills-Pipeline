# Installation / Getting Started

> Next.js 16.2.1 — https://nextjs.org/docs/app/getting-started/installation

## Quick Start

```bash
# pnpm
pnpm create next-app@latest my-app --yes && cd my-app && pnpm dev

# npm
npx create-next-app@latest my-app --yes && cd my-app && npm run dev

# yarn
yarn create next-app@latest my-app --yes && cd my-app && yarn dev

# bun
bun create next-app@latest my-app --yes && cd my-app && bun dev
```

`--yes` uses defaults: TypeScript, Tailwind CSS, ESLint, App Router, Turbopack, `@/*` import alias, and `AGENTS.md` / `CLAUDE.md` for coding agents. Visit `http://localhost:3000`.

---

## System Requirements

- Node.js **20.9+**
- macOS, Windows (including WSL), Linux

## Supported Browsers

- Chrome 111+, Edge 111+, Firefox 111+, Safari 16.4+

---

## CLI Installation (interactive)

```bash
npx create-next-app@latest
```

Default prompt offers: TypeScript, ESLint, Tailwind CSS, App Router, `AGENTS.md`.

Custom settings prompts:
- TypeScript: Yes/No
- Linter: ESLint / Biome / None
- React Compiler: Yes/No
- Tailwind CSS: Yes/No
- `src/` directory: Yes/No
- App Router: Yes/No (recommended: Yes)
- Import alias: `@/*` default

---

## Manual Installation

```bash
npm i next@latest react@latest react-dom@latest
```

Add scripts to `package.json`:
```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "eslint",
    "lint:fix": "eslint --fix"
  }
}
```

> Turbopack is now the default bundler (`next dev`). To use Webpack: `next dev --webpack`.

### Create the `app` directory

Create `app/layout.tsx` (required — must include `<html>` and `<body>`):

```tsx
export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  )
}
```

Create `app/page.tsx`:
```tsx
export default function Page() {
  return <h1>Hello, Next.js!</h1>
}
```

> If you forget the root layout, Next.js auto-creates it on `next dev`.
> Optionally use a [`src/`](https://nextjs.org/docs/app/api-reference/file-conventions/src-folder) directory to separate app code from config files.

### Create `public/` folder (optional)

Store static assets (images, fonts). Reference them from the base URL `/`:
```tsx
import Image from 'next/image'
export default function Page() {
  return <Image src="/profile.png" alt="Profile" width={100} height={100} />
}
```

---

## TypeScript Setup

Minimum version: **TypeScript v5.1.0**

Rename any file to `.ts` / `.tsx` and run `next dev` — Next.js auto-installs deps and creates `tsconfig.json`.

### VS Code TypeScript Plugin

Enable the built-in Next.js TypeScript plugin for advanced type-checking:
1. `Ctrl/⌘ + Shift + P` → "TypeScript: Select TypeScript Version"
2. Select "Use Workspace Version"

---

## Linting

Next.js 16 supports ESLint and Biome. `next build` no longer runs the linter automatically — run via npm scripts instead.

**ESLint:**
```json
{ "scripts": { "lint": "eslint", "lint:fix": "eslint --fix" } }
```

**Biome:**
```json
{ "scripts": { "lint": "biome check", "format": "biome format --write" } }
```

Migrate from `next lint` to ESLint CLI:
```bash
npx @next/codemod@canary next-lint-to-eslint-cli .
```

Use `eslint.config.mjs` (flat config, recommended in v16). See [ESLint API reference](https://nextjs.org/docs/app/api-reference/config/eslint).

---

## Absolute Imports and Path Aliases

Configure `tsconfig.json` / `jsconfig.json`:

```json
{
  "compilerOptions": {
    "baseUrl": "src/",
    "paths": {
      "@/styles/*": ["styles/*"],
      "@/components/*": ["components/*"]
    }
  }
}
```

```tsx
// Before
import { Button } from '../../../components/button'

// After
import { Button } from '@/components/button'
```

All `"paths"` values are relative to `baseUrl`.

---

## Getting Started Overview

The App Router Getting Started docs cover:
- Installation, Project Structure, Layouts and Pages, Linking and Navigating
- Server and Client Components, Fetching Data, Mutating Data
- Caching, Revalidating, Error Handling
- CSS, Image Optimization, Font Optimization, Metadata and OG Images
- Route Handlers, Proxy, Deploying, Upgrading