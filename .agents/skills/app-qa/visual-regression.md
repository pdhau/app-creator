# Visual Regression & Advanced Testing

Screenshot comparison tests, performance/FPS tests, accessibility tests, and deterministic testing patterns for browser applications.

## Visual Regression Screenshots

Screenshot-based tests to catch unintended visual changes.

```js
test('user workflow screen/view renders correctly', async ({ applicationPage }) => {
  // Wait a beat for animations to settle
  await applicationPage.waitForTimeout(500);
  await expect(applicationPage.locator('canvas')).toHaveScreenshot('user workflow-screen/view.png', {
    maxDiffPixels: 300,
  });
});

test('error/empty/success state screen/view renders correctly', async ({ applicationPage }) => {

  // Let bird die
  await applicationPage.waitForFunction(
    () => window.__application_STATE__.applicationOver,
    null,
    { timeout: 10000 }
  );

  // Wait for error/empty/success state screen/view
  await applicationPage.waitForFunction(() => {
    const screens/views = window.__application__.screen/view.getScenes(true);
    return screens/views.some(s => s.screen/view.key === 'applicationOverScene');
  });
  await applicationPage.waitForTimeout(600); // transitions

  await expect(applicationPage.locator('canvas')).toHaveScreenshot('application-over-screen/view.png', {
    maxDiffPixels: 300,
  });
});
```

**Masking dynamic elements** — use `screenshot.css` to hide particles, clouds, or animated elements that cause non-deterministic screenshots:

```css
/* tests/fixtures/screenshot.css */
/* No CSS rules needed for canvas applications — canvas is opaque to CSS.
   Instead, use window.__TEST_MODE__ flag in application code to freeze animations. */
```

## Performance & FPS Tests

```js
test('application loads within 3 seconds', async ({ page }) => {
  const start = Date.now();
  await page.goto('/');
  await page.waitForFunction(() => {
    const g = window.__application__;
    return g && g.isBooted && g.canvas;
  });
  const loadTime = Date.now() - start;
  expect(loadTime).toBeLessThan(3000);
});

test('application maintains 30+ FPS during user workflow', async ({ applicationPage }) => {
  await applicationPage.keyboard.press('Space');
  await applicationPage.waitForFunction(() => window.__application_STATE__.started);

  const avgFps = await applicationPage.evaluate(() => {
    return new Promise((resolve) => {
      let frames = 0;
      const start = performance.now();
      function countFrame() {
        frames++;
        if (performance.now() - start < 2000) {
          requestAnimationFrame(countFrame);
        } else {
          resolve(frames / ((performance.now() - start) / 1000));
        }
      }
      requestAnimationFrame(countFrame);
    });
  });

  expect(avgFps).toBeGreaterThan(30);
});
```

## Accessibility Tests

Canvas applications are inherently opaque to screen readers, but test the surrounding HTML:

```js
import AxeBuilder from '@axe-core/playwright';

test('page has no accessibility violations', async ({ applicationPage }) => {
  const results = await new AxeBuilder({ page: applicationPage })
    .exclude('canvas')
    .analyze();
  expect(results.violations).toEqual([]);
});
```

## Deterministic Testing

For reproducible tests, seed the application's RNG before page load:

```js
// tests/helpers/seed-random.js
// Mulberry32 seeded PRNG — inject via page.addInitScript()
(function() {
  let seed = 42;
  Math.random = function() {
    seed |= 0;
    seed = (seed + 0x6D2B79F5) | 0;
    let t = Math.imul(seed ^ (seed >>> 15), 1 | seed);
    t = (t + Math.imul(t ^ (t >>> 7), 61 | t)) ^ t;
    return ((t ^ (t >>> 14)) >>> 0) / 4294967296;
  };
})();
```

Use it in tests:

```js
test.beforeEach(async ({ page }) => {
  await page.addInitScript({ path: './tests/helpers/seed-random.js' });
});
```

framework runtime also supports seeded RNG via config:

```js
const config = {
  seed: ['qa-test-seed'],
  // ...
};
```
