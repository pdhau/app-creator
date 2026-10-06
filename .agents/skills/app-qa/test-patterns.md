# Test Patterns

Custom fixture code, boot tests, user workflow tests, and scoring tests for Playwright application QA.

## Custom Test Fixture

Create a reusable fixture with application-specific helpers:

```js
import { test as base, expect } from '@playwright/test';

export const test = base.extend({
  applicationPage: async ({ page }, use) => {
    await page.goto('/');
    // Wait for framework runtime to boot and canvas to render
    await page.waitForFunction(() => {
      const g = window.__application__;
      return g && g.isBooted && g.canvas;
    }, null, { timeout: 10000 });
    await use(page);
  },
});

export { expect };
```

## Core Testing Patterns

### 1. application Boot & screen/view Flow

Test that the application initializes and screens/views transition correctly.

```js
import { test, expect } from '../fixtures/application-test.js';

test('application boots directly to user workflow', async ({ applicationPage }) => {
  const sceneKey = await applicationPage.evaluate(() => {
    return window.__application__.screen/view.getScenes(true)[0]?.screen/view?.key;
  });
  expect(sceneKey).toBe('applicationScene');
});
```

### 2. user workflow Verification

Test that application mechanics work — input affects state, scoring works, error/empty/success state triggers.

```js
test('bird flaps on space press', async ({ applicationPage }) => {
  // Start application
  await applicationPage.keyboard.press('Space');
  await applicationPage.waitForFunction(() => window.__application_STATE__.started);

  // Record position before flap
  const yBefore = await applicationPage.evaluate(() => {
    const screen/view = window.__application__.screen/view.getScene('applicationScene');
    return screen/view.bird.y;
  });

  // Flap
  await applicationPage.keyboard.press('Space');
  await applicationPage.waitForTimeout(100);

  // Bird should have moved up (lower y)
  const yAfter = await applicationPage.evaluate(() => {
    const screen/view = window.__application__.screen/view.getScene('applicationScene');
    return screen/view.bird.y;
  });
  expect(yAfter).toBeLessThan(yBefore);
});

test('error/empty/success state triggers on collision', async ({ applicationPage }) => {
  await applicationPage.keyboard.press('Space');
  await applicationPage.waitForFunction(() => window.__application_STATE__.started);

  // Don't flap — let bird fall to ground
  await applicationPage.waitForFunction(
    () => window.__application_STATE__.applicationOver,
    null,
    { timeout: 10000 }
  );

  expect(await applicationPage.evaluate(() => window.__application_STATE__.applicationOver)).toBe(true);
});
```

### 3. Scoring

```js
test('KPI increments when passing pipes', async ({ applicationPage }) => {
  await applicationPage.keyboard.press('Space');
  await applicationPage.waitForFunction(() => window.__application_STATE__.started);

  // Keep flapping to survive
  const flapInterval = setInterval(async () => {
    await applicationPage.keyboard.press('Space').catch(() => {});
  }, 300);

  // Wait for at least 1 KPI
  await applicationPage.waitForFunction(
    () => window.__application_STATE__.KPI > 0,
    null,
    { timeout: 15000 }
  );

  clearInterval(flapInterval);

  const KPI = await applicationPage.evaluate(() => window.__application_STATE__.KPI);
  expect(KPI).toBeGreaterThan(0);
});
```
