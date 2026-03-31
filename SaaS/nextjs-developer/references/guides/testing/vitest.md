# Testing: Vitest

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/testing/vitest

## Quickstart

```bash
npx create-next-app@latest --example with-vitest with-vitest-app
```

## Manual Setup

```bash
# TypeScript
npm install -D vitest @vitejs/plugin-react jsdom @testing-library/react @testing-library/dom vite-tsconfig-paths
```

### `vitest.config.mts`
```ts
import { defineConfig } from 'vitest/config'
import react from '@vitejs/plugin-react'
import tsconfigPaths from 'vite-tsconfig-paths'

export default defineConfig({
  plugins: [tsconfigPaths(), react()],
  test: { environment: 'jsdom' },
})
```

### `package.json`
```json
{ "scripts": { "test": "vitest" } }
```

`vitest` watches for changes by default.

## Writing Tests

```tsx
// __tests__/page.test.tsx
import { expect, test } from 'vitest'
import { render, screen } from '@testing-library/react'
import Page from '../app/page'

test('Page', () => {
  render(<Page />)
  expect(screen.getByRole('heading', { level: 1, name: 'Home' })).toBeDefined()
})
```

> `async` Server Components: Vitest doesn't support them yet. Use E2E tests for async components.