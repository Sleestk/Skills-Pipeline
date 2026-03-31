# Next.js Compiler (v16.2.1)

The Next.js Compiler is written in Rust using [SWC](https://swc.rs/). It transforms and minifies JavaScript for production — replacing Babel (for individual files) and Terser (for bundle minification).

- **17x faster** than Babel
- Default since Next.js v12
- Falls back to Babel automatically if a `.babelrc` file exists or unsupported features are used

---

## Why SWC?

- **Extensibility** — usable as a Crate inside Next.js without forking
- **Performance** — ~3x faster Fast Refresh, ~5x faster builds vs Babel
- **WebAssembly** — Rust's WASM support enables cross-platform Next.js
- **Community** — growing Rust ecosystem

---

## Supported Features

### Styled Components

```js
// next.config.js
module.exports = {
  compiler: {
    styledComponents: true,
    // or with options:
    styledComponents: {
      displayName: boolean,     // default: true in dev, false in prod
      ssr: boolean,             // default: true
      fileName: boolean,        // default: true
      topLevelImportPaths: [],
      meaninglessFileNames: ['index'],
      minify: boolean,          // default: true
      transpileTemplateLiterals: boolean, // default: true
      namespace: '',
      pure: boolean,            // default: false
      cssProp: boolean,         // default: true
    },
  },
}
```

`ssr` and `displayName` are the main requirements for `styled-components` in Next.js.

---

### Jest

Simplifies Jest + Next.js config: auto-mocks CSS/image imports, sets up SWC transforms, loads `.env` files, ignores `node_modules` and `.next`.

```js
// jest.config.js
const nextJest = require('next/jest')
const createJestConfig = nextJest({ dir: './' })
const customJestConfig = {
  setupFilesAfterEnv: ['<rootDir>/jest.setup.js'],
}
module.exports = createJestConfig(customJestConfig)
```

---

### Relay

```js
// next.config.js
module.exports = {
  compiler: {
    relay: {
      src: './',
      artifactDirectory: './__generated__',  // must be outside pages/
      language: 'typescript',
      eagerEsModules: false,
    },
  },
}
```

> `artifactDirectory` must be outside `pages/` — files inside `pages/` are treated as routes.

---

### Remove React Properties

Removes JSX properties matching a regex. Similar to `babel-plugin-react-remove-properties`.

```js
// Remove properties matching default regex ^data-test
module.exports = { compiler: { reactRemoveProperties: true } }

// Custom regex (Rust regex syntax)
module.exports = { compiler: { reactRemoveProperties: { properties: ['^data-custom$'] } } }
```

---

### Remove Console

```js
// Remove all console.* calls
module.exports = { compiler: { removeConsole: true } }

// Remove all except console.error
module.exports = { compiler: { removeConsole: { exclude: ['error'] } } }
```

Applies to application code only, not `node_modules`.

---

### Legacy Decorators

Auto-detected from `jsconfig.json` or `tsconfig.json`. For compatibility with older libraries like MobX only — not recommended for new code.

```json
{ "compilerOptions": { "experimentalDecorators": true } }
```

---

### importSource (jsxImportSource)

Auto-detected from `jsconfig.json` or `tsconfig.json`. Used with libraries like Theme UI.

```json
{ "compilerOptions": { "jsxImportSource": "theme-ui" } }
```

---

### Emotion

```js
// next.config.js
module.exports = {
  compiler: {
    emotion: boolean | {
      sourceMap?: boolean,       // default: true
      autoLabel?: 'never' | 'dev-only' | 'always',  // default: 'dev-only'
      labelFormat?: string,      // default: '[local]'. Supports [local], [filename], [dirname]
      importMap?: { [packageName]: { [exportName]: { canonicalImport?, styledBaseImport? } } },
    },
  },
}
```

---

### Minification

SWC minifier is the default since v13 — **7x faster than Terser**.

> As of v15, `swcMinify` flag is removed. Minification is no longer customizable via `next.config.js`.

---

### Module Transpilation

Transpiles and bundles dependencies from local packages (monorepos) or `node_modules`. Replaces the old `next-transpile-modules` package.

```js
module.exports = { transpilePackages: ['@acme/ui', 'lodash-es'] }
```

---

### Define (Build-time variable replacement)

Statically replaces variables at build time.

```js
module.exports = {
  compiler: {
    define: {
      MY_VARIABLE: 'my-string',
      'process.env.MY_ENV_VAR': 'my-env-var',
    },
    defineServer: {
      MY_SERVER_VARIABLE: 'my-server-var',  // server + edge only
    },
  },
}
```

---

### Build Lifecycle Hooks

#### `runAfterProductionCompile`

Runs after production compilation, before type checking and static page generation. Useful for third-party tools collecting build outputs (e.g. sourcemaps).

```js
module.exports = {
  compiler: {
    runAfterProductionCompile: async ({ distDir, projectDir }) => {
      // distDir: build output dir (default: .next)
      // projectDir: project root
    },
  },
}
```

---

## Experimental Features

### SWC Trace Profiling

```js
module.exports = { experimental: { swcTraceProfiling: true } }
```

Generates `swc-trace-profile-${timestamp}.json` in `.next/`. Open in Chrome DevTools (`chrome://tracing/`), Perfetto (`ui.perfetto.dev`), or Speedscope (`speedscope.app`).

### SWC Plugins

```js
module.exports = {
  experimental: {
    swcPlugins: [
      ['plugin-name', { ...pluginOptions }],
    ],
  },
}
```

Accepts array of `[pluginPath, options]` tuples. `pluginPath` can be an npm package name or an absolute path to a `.wasm` binary.

---

## Unsupported Features / Babel Fallback

If a `.babelrc` file is present, Next.js **automatically falls back to Babel** for individual file transforms. This preserves backwards compatibility with custom Babel plugins.

---

## Version History

| Version | Change |
|---|---|
| v13.1.0 | Module Transpilation and Modularize Imports stable |
| v13.0.0 | SWC Minifier default |
| v12.3.0 | SWC Minifier stable |
| v12.2.0 | SWC Plugins experimental support |
| v12.1.0 | Added Styled Components, Jest, Relay, Remove React Properties, Legacy Decorators, Remove Console, jsxImportSource |
| v12.0.0 | Next.js Compiler introduced |