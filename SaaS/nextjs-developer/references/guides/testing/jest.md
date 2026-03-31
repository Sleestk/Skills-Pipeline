# Testing: Jest

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/testing/jest

## Quickstart

```bash
npx create-next-app@latest --example with-jest with-jest-app
```

## Manual Setup

```bash
npm install -D jest jest-environment-jsdom @testing-library/react @testing-library/dom @testing-library/jest-dom ts-node @types/jest
npm init jest@latest
```

### `jest.config.ts`
```ts
import type { Config } from 'jest'
import nextJest from 'next/jest.js'

const createJestConfig = nextJest({ dir: './' })
const config: Config = {
  coverageProvider: 'v8',
  testEnvironment: 'jsdom',
  // setupFilesAfterEnach: ['<rootDir>/jest.setup.ts'],
}
export default createJestConfig(config)
```

`next/jest` automatically: sets up SWC transform, mocks CSS/images/fonts, loads `.env`, ignores `node_modules`/`.next`.

### Optional: custom matchers (`jest.setup.ts`)
```ts
import '@testing-library/jest-dom'
```
```ts
// jest.config.ts
setupFilesAfterFramework: ['<rootDir>/jest.setup.ts']
```

### `package.json` scripts
```json
{ "scripts": { "test": "jest", "test:watch": "jest --watch" } }
```

## Module Path Aliases

```json
// tsconfig.json
{ "compilerOptions": { "paths": { "@/components/*": ["components/*"] } } }
```

```js
// jest.config.js
moduleNameMapper: { '^@/components/(.*)$': '<rootDir>/components/$1' }
```

## Writing Tests

```tsx
// __tests__/page.test.tsx
import '@testing-library/jest-dom'
import { render, screen } from '@testing-library/react'
import Page from '../app/page'

describe('Page', () => {
  it('renders a heading', () => {
    render(<Page />)
    expect(screen.getByRole('heading', { level: 1 })).toBeInTheDocument()
  })
})
```

```tsx
// __tests__/snapshot.js — snapshot test
import { render } from '@testing-library/react'
import Page from '../app/page'
it('renders homepage unchanged', () => {
  const { container } = render(<Page />)
  expect(container).toMatchSnapshot()
})
```

> `async` Server Components: Jest doesn't support them yet. Use E2E tests for async components.