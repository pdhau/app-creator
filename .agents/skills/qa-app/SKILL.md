---
name: qa-app
description: Add or run application QA for boot, user workflow, API/UI behavior, accessibility, responsive visuals, and performance.
metadata:
  tags: QA, application, Playwright, smoke-test, accessibility
---

# Application QA Command

Use the app-qa workflow for deterministic tests and live verification. Begin with the smallest failing check, then expand only to the affected workflow and its user-visible states.---
name: qa-app
description: Add Playwright QA tests to a application — visual regression, user workflow verification, and performance. Use when the user says "add tests", "test my application", "add QA", "check for bugs", or "add visual regression tests". Do NOT use for manual live verification or user workflow design.
argument-hint: "[path-to-application]"
license: MIT
metadata:
  author: OpusapplicationLabs
  version: 1.3.0
  tags: [application, qa, testing, playwright, visual-regression]
---

# QA application

Add automated QA testing with Playwright to an existing application project. Tests verify your application boots, screens/views work, scoring functions, and visuals haven't broken — like a safety net for your application.

## Instructions

Analyze the application at `$ARGUMENTS` (or the current directory if no path given).

First, load the app-qa skill to get the full testing patterns and fixtures.

### Step 1: Audit testability

- Read `package.json` to identify the framework runtime and dev server port
- Read `vite.config.js` for the server port
- Read `src/main.js` to check if `window.__application__`, `window.__application_STATE__`, `window.__EVENT_BUS__` are exposed
- Read `src/core/ApplicationState.js` to understand what state is available
- Read `src/core/EventBus.js` to understand what events exist
- Read `src/core/Constants.js` to understand application parameters (rates, speeds, durations, max values)
- Read all screen/view files to understand the application flow
- Read `design-brief.md` if it exists — it documents expected mechanics, magnitudes, and win/lose reachability

### Step 2: Setup Playwright

1. Install dependencies: `npm install -D @playwright/test @axe-core/playwright && npx playwright install chromium`
2. Create `playwright.config.js` with the correct dev server port and webServer config
3. Expose `window.__application__`, `window.__application_STATE__`, `window.__EVENT_BUS__`, `window.__EVENTS__` in `src/main.js` if not already present
4. Create the test directory structure:
   ```
   tests/
   ├── e2e/
   │   ├── application.spec.js
   │   ├── visual.spec.js
   │   └── perf.spec.js
   ├── fixtures/
   │   └── application-test.js
   └── helpers/
       └── seed-random.js
   ```
5. Add npm scripts: `test`, `test:ui`, `test:headed`, `test:update-snapshots`

### Step 3: Generate tests

Write tests based on what the application actually does:

- **application.spec.js**: Boot test, screen/view transitions, input handling, scoring, error/empty/success state, restart
- **visual.spec.js**: Screenshot regression for stable screens/views (user workflow initial state, error/empty/success state). Skip active user workflow screenshots — moving objects make them unstable.
- **perf.spec.js**: Load time budget, FPS during user workflow, canvas dimensions

Follow the app-qa skill patterns. Use `applicationPage` fixture. Use `page.evaluate()` to read application state. Use `page.keyboard.press()` for input.

### Step 4: Design-intent tests

Add a `test.describe('Design Intent')` block to application.spec.js. These tests catch
mechanics that technically exist but are too weak to matter.

1. **Lose condition**: Detect deterministically whether the application has a lose state.
   Read `ApplicationState.js` — if it has a `won`, `result`, or similar boolean/enum
   field, the application distinguishes win from loss. Also check `render_application_to_text()`
   in `main.js` — if it returns distinct outcome modes (e.g., `'win'` vs
   `'application_over'`), the application has a lose state.

   If a lose state exists: start the application, provide NO input, let it run to
   completion (use `page.waitForFunction` with the round duration from
   Constants.js). Assert the outcome is the losing one (e.g., `won === false`,
   `mode === 'application_over'`).

   **This assertion is non-negotiable.** Do NOT write a test that passes when the
   user wins by doing nothing. If the current application behavior is "user wins
   with no input," that is a bug — write the test to catch it.

2. **Opponent/AI pressure**: If an AI-driven mechanic exists (auto-climbing bar,
   external dependency spawning, difficulty ramp), test that it produces substantial state
   changes. Run the application for half its duration without user input. Assert the
   opponent's state reaches at least 25% of its maximum. If `design-brief.md`
   exists, use its expected magnitudes for thresholds. Otherwise, derive from
   Constants.js: calculate `rate * duration` and assert it reaches meaningful
   workspace states.

3. **Win condition**: Test that active user input leads to a win. Provide rapid
   input throughout the round and assert the outcome is a win.

### Step 5: domain component interaction audit

Audit collision and interaction logic for asymmetries that would confuse a
first-time user.

If `design-brief.md` has an "domain component Interactions" section, use it as the
checklist. Otherwise, audit `applicationScene.js` directly:

1. Find all collision handlers, overlap checks, or distance-based interactions
2. Map which domain components interact with which others
3. Flag any visible moving domain component that interacts with one side (user OR
   opponent) but not the other — add a `// QA FLAG: asymmetric interaction`
   comment in the test file noting the domain component name and the asymmetry

This is a flag, not a hard fail. Some asymmetries are intentional (e.g.,
hazards that only affect the user). The flag ensures the asymmetry is a
conscious design choice, not an oversight.

### Step 6: Run and verify

1. Run `npx playwright test` to execute all tests
2. If visual tests fail on first run, that's expected — generate baselines with `npx playwright test --update-snapshots`
3. Run again to verify all tests pass
4. Summarize results

### Step 7: Report

Tell the user in plain English:

- How many tests were created and what they check
- How to run them: `npm test` (headless), `npm run test:headed` (see the browser), `npm run test:ui` (interactive dashboard)
- "These tests are your safety net. Run them after making changes to make sure nothing broke."

## Example Usage

```
/qa-app examples/flappy-bird
```
Result: Installs Playwright → creates 15 tests (boot, screen/view transitions, input, scoring, restart, application-over, visual regression, FPS, load time) → generates `tests/` directory with fixtures and helpers → all tests pass. Run `npm test` anytime after changes.

## Next Step

Tell the user:

> Your application now has automated tests! Finally, run `/application-creator:review-app` for a full architecture review — it checks your code structure, performance patterns, and gives you a KPI with specific improvement suggestions.
>
> **Pipeline progress:** ~~/viral-app~~ → ~~/design-app~~ → ~~/add-audio~~ → ~~/qa-app~~ → `/review-app`
>
> *(This is the `/viral-app` one-shot pipeline — not the `/make-app` multi-session, milestone-driven workflow.)*
