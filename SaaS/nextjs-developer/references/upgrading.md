# Upgrading

> Next.js 16.2.1 — https://nextjs.org/docs/app/getting-started/upgrading

---

## Latest Version

### Using the `upgrade` command (v16.1.0+)

```bash
# pnpm
pnpm next upgrade

# npm
npx next upgrade

# yarn
yarn next upgrade

# bun
bunx next upgrade
```

### Older versions (before v16.1.0)

```bash
npx @next/codemod@canary upgrade latest
```

### Manual upgrade

```bash
# pnpm
pnpm i next@latest react@latest react-dom@latest eslint-config-next@latest

# npm
npm i next@latest react@latest react-dom@latest eslint-config-next@latest

# yarn
yarn add next@latest react@latest react-dom@latest eslint-config-next@latest

# bun
bun add next@latest react@latest react-dom@latest eslint-config-next@latest
```

---

## Canary Version

Make sure the latest stable version is working, then:

```bash
pnpm add next@canary   # or npm/yarn/bun equivalent
```

### Features currently in canary

**Authentication:**
- `forbidden()` / `unauthorized()` functions
- `forbidden.js` / `unauthorized.js` file conventions
- `authInterrupts` config option

---

## Version Guides

For in-depth upgrade instructions including breaking changes and codemods:

- **[Version 16](https://nextjs.org/docs/app/guides/upgrading/version-16)** — Upgrade from v15 to v16
- **[Version 15](https://nextjs.org/docs/app/guides/upgrading/version-15)** — Upgrade from v14 to v15
- **[Version 14](https://nextjs.org/docs/app/guides/upgrading/version-14)** — Upgrade from v13 to v14