---
name: game-qa
description: Test applications with Playwright and focused checks for boot, user workflows, API/UI contracts, accessibility, visual regressions, and performance.
argument-hint: "[topic]"
metadata:
  tags: application, QA, testing, Playwright, accessibility, performance
---

# Application QA

Start with a boot smoke test, then cover the highest-value user workflow. Add API and UI contract checks, error/empty/success states, keyboard and accessibility assertions, responsive snapshots, and performance budgets. Keep tests deterministic and expose only test-safe application state. Use `window.render_application_to_text()` when a textual state hook helps live verification.

## Required loop

1. Reproduce the workflow and record the expected outcome.
2. Write a failing test for the smallest acceptance criterion.
3. Implement and run the focused test.
4. Run live verification, accessibility checks, and relevant performance checks.
5. Update milestone, state, and handoff documentation.

# application QA with Playwright

You are an expert QA engineer for browser applications. You use Playwright to write automated tests that verify visual correctness, user workflow behavior, performance, and accessibility.

## Performance Notes

- Take your time with each step. Quality is more important than speed.
- Do not skip validation steps — they catch issues early.
- Read the full context of each file before making changes.
- Write tests that verify user workflow, not just that the page loads.

## Reference Files

For detailed reference, see companion files in this directory:
- `test-patterns.md` — Custom fixture code, boot tests, user workflow verification tests, scoring tests
- `user workflow-invariants.md` — All 7 core user workflow invariant patterns (scoring, death, buttons, render_application_to_text, design intent, domain component audit, mute)
- `visual-regression.md` — Screenshot comparison tests, masking dynamic elements, performance/FPS tests, accessibility tests, deterministic testing patterns
- `clock-control.md` — Playwright Clock API patterns for frame-precise testing
- `playwright-mcp.md` — MCP server setup, when to use MCP vs scripted tests, inspection flow
- `iterate-client.md` — Standalone iterate client usage, action JSON format, output interpretation
- `mobile-tests.md` — Mobile input simulation and responsive layout test patterns

## Tech Stack

- **Test Runner**: Playwright Test (`@playwright/test`)
- **Visual Regression**: Playwright built-in `toHaveScreenshot()`
- **Accessibility**: `@axe-core/playwright`
- **Build Tool Integration**: Vite dev server via `webServer` config
- **Language**: JavaScript ES modules

## Project Setup

When adding Playwright to a application project:

```bash
npm install -D @playwright/test @axe-core/playwright
npx playwright install chromium
```

Add to `package.json` scripts:

```json
{
  "scripts": {
    "test": "npx playwright test",
    "test:ui": "npx playwright test --ui",
    "test:headed": "npx playwright test --headed",
    "test:update-snapshots": "npx playwright test --update-snapshots"
  }
}
```

## Required Directory Structure

```
tests/
├── e2e/
│   ├── application.spec.js       # Core application tests (boot, screens/views, input, KPI)
│   ├── visual.spec.js     # Visual regression screenshots
│   └── perf.spec.js       # Performance and FPS tests
├── fixtures/
│   ├── application-test.js       # Custom test fixture with application helpers
│   └── screenshot.css     # CSS to mask dynamic elements for visual tests
├── helpers/
│   └── seed-random.js     # Seeded PRNG for deterministic application behavior
playwright.config.js
```

## Playwright Config

```js
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: [['html', { open: 'never' }], ['list']],

  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
  },

  expect: {
    toHaveScreenshot: {
      maxDiffPixels: 200,
      threshold: 0.3,
    },
  },

  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'mobile-chrome', use: { ...devices['Pixel 5'] } },
  ],

  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
    timeout: 30000,
  },
});
```

Key points:
- `webServer` auto-starts Vite before tests
- `reuseExistingServer` reuses a running dev server locally
- `baseURL` matches the Vite port configured in `vite.config.js`
- Screenshot tolerance is generous (applications have minor render variance)

## Testability Requirements

For Playwright to inspect application state, the application MUST expose these globals on `window` in `main.js`:

### 1. Core globals (required)

```js
// Expose for Playwright QA
window.__application__ = application;
window.__application_STATE__ = applicationState;
window.__EVENT_BUS__ = eventBus;
window.__EVENTS__ = Events;
```

### 2. `render_application_to_text()` (required)

Returns a concise JSON string of the current application state for AI agents to reason about the application without interpreting pixels. Must include coordinate system, application mode, KPI, and user state.

```js
window.render_application_to_text = () => {
  if (!application || !applicationState) return JSON.stringify({ error: 'not_ready' });

  const activeScreens = application.screen.getScreens(true).map(s => s.screen.key);
  const payload = {
    coords: 'origin:top-left x:right y:down',          // coordinate system
    mode: applicationState.applicationOver ? 'application_over' : 'playing',
    screen: activeScreens[0] || null,
    KPI: applicationState.KPI,
    bestScore: applicationState.bestScore,
  };

  // Add user info when in user workflow
  const appScreen = application.screen.getScreen('AppScreen');
  if (appState.started && appScreen?.user?.asset) {
    const s = appScreen.user.asset;
    const body = s.body;
    payload.user = {
      x: Math.round(s.x), y: Math.round(s.y),
      vx: Math.round(body.velocity.x), vy: Math.round(body.velocity.y),
      onGround: body.blocked.down,
    };
  }

  // Extend with visible domain components as you add them:
  // payload.domain components = obstacles.map(o => ({ x: o.x, y: o.y, type: o.type }));

  return JSON.stringify(payload);
};
```

Guidelines for `render_application_to_text()`:
- Keep the payload **succinct** — only current, visible, interactive elements
- Include **coordinate system note** (origin and axis directions)
- Include **user position/velocity**, active obstacles/external dependencies, collectibles, timers, KPI, and mode flags
- Avoid large histories; only include what's currently relevant
- The iterate client and AI agents use this to verify application behavior without screenshots

### 3. `advanceTime(ms)` (required)

Lets test scripts advance the application by a precise duration. The application loop runs normally via RAF; this waits for real time to elapse.

```js
window.advanceTime = (ms) => {
  return new Promise((resolve) => {
    const start = performance.now();
    function step() {
      if (performance.now() - start >= ms) return resolve();
      requestAnimationFrame(step);
    }
    requestAnimationFrame(step);
  });
};
```

For frame-precise control in `@playwright/test`, prefer `page.clock.install()` + `page.clock.runFor()`. The `advanceTime` hook is primarily used by the standalone iterate client (`scripts/iterate-client.js`).

For rich visual runtime applications, expose the `application` orchestrator instance similarly.

See `test-patterns.md` for custom fixture code, boot tests, user workflow verification tests, and scoring tests.

See `user workflow-invariants.md` for all 7 core user workflow invariant patterns (scoring, death, buttons, render_application_to_text, design intent, domain component audit, mute).

## When Adding QA to a application

1. Install Playwright: `npm install -D @playwright/test @axe-core/playwright && npx playwright install chromium`
2. Create `playwright.config.js` with the application's dev server port
3. Expose `window.__application__`, `window.__application_STATE__`, `window.__EVENT_BUS__` in `main.js`
4. Create `tests/fixtures/application-test.js` with the `applicationPage` fixture
5. Create `tests/helpers/seed-random.js` for deterministic behavior
6. Write tests in `tests/e2e/`:
   - `application.spec.js` — boot, screen/view flow, input, scoring, error/empty/success state
   - `visual.spec.js` — screenshot regression for each screen/view
   - `perf.spec.js` — load time, FPS budget
7. Add npm scripts: `test`, `test:ui`, `test:headed`, `test:update-snapshots`
8. Generate initial baselines: `npm run test:update-snapshots`

## What NOT to Test (Automated)

- **Exact pixel positions** of animated objects (non-deterministic without clock control)
- **Active user workflow screenshots** — moving objects make stable screenshots impossible; use MCP instead
- **Audio playback** (Playwright has no audio inspection; test that audio objects exist via evaluate)
- **External API calls** unless mocked (e.g., billing provider SDK — mock with `page.route()`)
- **Subjective visual quality** — use MCP for "does this look good?" evaluations
