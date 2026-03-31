# Testing: Playwright

> Next.js 16.2.1 — https://nextjs.org/docs/app/guides/testing/playwright

## Quickstart

```bash
npx create-next-app@latest --example with-playwright with-playwright-app
```

## Manual Setup

```bash
npm init playwright
```

Creates `playwright.config.ts` and installs browsers.

## E2E Test Example

```ts
// tests/example.spec.ts
import { test, expect } from '@playwright/test'

test('should navigate to the about page', async ({ page }) => {
  await page.goto('http://localhost:3000/')
  await page.click('text=About')
  await expect(page).toHaveURL('http://localhost:3000/about')
  await expect(page.locator('h1')).toContainText('About')
})
```

Add `baseURL: 'http://localhost:3000'` to `playwright.config.ts` to use `page.goto('/')`.

## Running Tests

Run against **production build** (closest to real behavior):
```bash
npm run build && npm run start
# In another terminal:
npx playwright test
```

Or use `webServer` in `playwright.config.ts` to auto-start the dev server.

## CI (headless)

```bash
npx playwright install-deps  # install browser dependencies
npx playwright test
```

## Activity/`display:none` Content

Playwright's `getByRole` automatically filters by visibility — recommended for tests involving Activity boundaries or conditional rendering:

```ts
// Good — filters by visibility
await page.getByRole('button', { name: 'Submit' }).click()
// Fallback — explicit filter
await page.locator('.card').filter({ visible: true }).first().click()
```