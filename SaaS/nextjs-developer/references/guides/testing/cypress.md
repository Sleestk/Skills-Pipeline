# Testing: Cypress

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/testing/cypress

## Quickstart

```bash
npx create-next-app@latest --example with-cypress with-cypress-app
```

## Manual Setup

```bash
npm install -D cypress
```

```json
{ "scripts": { "cypress:open": "cypress open" } }
```

Run `npm run cypress:open` → choose E2E or Component Testing → auto-creates `cypress.config.js`.

## E2E Test Example

```js
// cypress/e2e/app.cy.js
describe('Navigation', () => {
  it('should navigate to the about page', () => {
    cy.visit('http://localhost:3000/')
    cy.get('a[href*="about"]').click()
    cy.url().should('include', '/about')
    cy.get('h1').contains('About')
  })
})
```

**Run against production build**: `npm run build && npm run start`, then `npm run cypress:open`.

Config tip: add `baseUrl: 'http://localhost:3000'` to use `cy.visit('/')` instead of full URL.

## Component Test Example

```ts
// cypress.config.ts
export default defineConfig({
  component: { devServer: { framework: 'next', bundler: 'webpack' } }
})
```

```tsx
// cypress/component/about.cy.tsx
import Page from '../../app/page'
describe('<Page />', () => {
  it('renders expected content', () => {
    cy.mount(<Page />)
    cy.get('h1').contains('Home')
    cy.get('a[href="/about"]').should('be.visible')
  })
})
```

> Cypress doesn't support `async` Server Components in component tests — use E2E for those.

## CI (headless)

```json
{
  "scripts": {
    "e2e:headless": "start-server-and-test dev http://localhost:3000 \"cypress run --e2e\"",
    "component:headless": "cypress run --component"
  }
}
```