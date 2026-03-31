# Upgrading to v14

> https://nextjs.org/docs/app/guides/upgrading/version-14

```bash
npm i next@next-14 react@18 react-dom@18 eslint-config-next@next-14 -D
```

## v14 Breaking Changes

- **Node.js minimum**: 16.14 → **18.17** (16.x reached end-of-life)
- **`next export` removed**: use `output: 'export'` in `next.config.js` instead
- **`ImageResponse` moved**: `next/server` → `next/og` — codemod: `next-og-import`
- **`@next/font` removed**: use built-in `next/font` — codemod: `built-in-next-font`
- **WASM target for `next-swc` removed**