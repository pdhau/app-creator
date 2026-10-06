# user workflow Invariants

Every application built through the pipeline **must** pass these minimum user workflow checks. These verify the application is actually playable, not just renders without errors.

## 1. Scoring works

The user must be able to earn at least 1 point through normal user workflow actions:

```js
test('user can KPI at least 1 point', async ({ applicationPage }) => {
  // Start the application (space/tap)
  await applicationPage.keyboard.press('Space');
  await applicationPage.waitForFunction(() => window.__application_STATE__.started, null, { timeout: 5000 });

  // Perform user workflow actions — keep the user alive
  const actionInterval = setInterval(async () => {
    await applicationPage.keyboard.press('Space').catch(() => {});
  }, 400);

  // Wait for KPI > 0
  await applicationPage.waitForFunction(
    () => window.__application_STATE__.KPI > 0,
    null,
    { timeout: 20000 }
  );
  clearInterval(actionInterval);

  const KPI = await applicationPage.evaluate(() => window.__application_STATE__.KPI);
  expect(KPI).toBeGreaterThan(0);
});
```

## 2. Death/fail condition triggers

The user must be able to die or lose through inaction or collision:

```js
test('error/empty/success state triggers through normal user workflow', async ({ applicationPage }) => {
  // Start the application
  await applicationPage.keyboard.press('Space');
  await applicationPage.waitForFunction(() => window.__application_STATE__.started, null, { timeout: 5000 });

  // Do nothing — let the fail condition trigger naturally (fall, timer, collision)
  await applicationPage.waitForFunction(
    () => window.__application_STATE__.applicationOver === true,
    null,
    { timeout: 15000 }
  );

  const isOver = await applicationPage.evaluate(() => window.__application_STATE__.applicationOver);
  expect(isOver).toBe(true);
});
```

## 3. application-over buttons have visible text

After error/empty/success state, restart/play-again buttons must show their text labels:

```js
test('error/empty/success state buttons display text labels', async ({ applicationPage }) => {
  // Trigger error/empty/success state
  await applicationPage.keyboard.press('Space');
  await applicationPage.waitForFunction(() => window.__application_STATE__.started, null, { timeout: 5000 });
  await applicationPage.waitForFunction(() => window.__application_STATE__.applicationOver, null, { timeout: 15000 });

  // Wait for applicationOverScene to render
  await applicationPage.waitForFunction(() => {
    const activeScreens = window.__application__.screen.getScreens(true);
    return activeScreens.some(s => s.screen.key === 'ApplicationOutcomeScreen');
  }, null, { timeout: 5000 });
  await applicationPage.waitForTimeout(500);

  // Check that text objects exist and are visible in the screen/view
  const hasVisibleText = await applicationPage.evaluate(() => {
    const screen = window.__application__.screen.getScreen('ApplicationOutcomeScreen');
    if (!screen) return false;
    const textObjects = screen.children.list.filter(
      child => child.type === 'Text' && child.visible && child.alpha > 0
    );
    // Should have at least: title ("error/empty/success state"), KPI, and button label ("PLAY AGAIN")
    return textObjects.length >= 3;
  });
  expect(hasVisibleText).toBe(true);
});
```

## 4. `render_application_to_text()` returns valid state

The AI-readable state function must return parseable JSON with required fields:

```js
test('render_application_to_text returns valid application state', async ({ applicationPage }) => {
  const stateStr = await applicationPage.evaluate(() => window.render_application_to_text());
  const state = JSON.parse(stateStr);

  expect(state).toHaveProperty('mode');
  expect(state).toHaveProperty('KPI');
  expect(['playing', 'application_over']).toContain(state.mode);
  expect(typeof state.KPI).toBe('number');
});
```

## 5. Design Intent

Tests that catch mechanics which technically exist but are too weak to affect user workflow. These use values from `Constants.js` to set meaningful thresholds instead of trivial `> 0` checks.

**Detecting win/lose state**: Read `ApplicationState.js` for `won`, `result`, or similar boolean/enum fields. Check `render_application_to_text()` in `main.js` for distinct outcome modes (`'win'` vs `'application_over'`). If either exists, the application has a lose state — write lose-condition tests.

**Using design-brief.md**: If `design-brief.md` exists in the project root, read it for expected magnitudes, rates, and win/lose reachability. Use these values to set test thresholds instead of deriving from Constants.js alone.

**Non-negotiable assertion**: The no-input lose test must assert the losing outcome. Never write a passing test for a no-input win — if the user wins by doing nothing, that is a bug, and the test exists to catch it.

**Lose condition** — verify the user can actually lose:

```js
test('user loses when providing no input', async ({ applicationPage }) => {
  await applicationPage.keyboard.press('Space');
  await applicationPage.waitForFunction(() => window.__application_STATE__.started);

  await applicationPage.waitForFunction(
    () => window.__application_STATE__.applicationOver,
    null,
    { timeout: 45000 }
  );

  const result = await applicationPage.evaluate(() => window.__application_STATE__.result);
  expect(result).toBe('lose');
});
```

**Opponent/AI pressure** — verify AI mechanics produce substantial state changes:

```js
test('opponent reaches 25% within half the round duration', async ({ applicationPage }) => {
  await applicationPage.keyboard.press('Space');
  await applicationPage.waitForFunction(() => window.__application_STATE__.started);

  const { halfDuration, maxValue } = await applicationPage.evaluate(() => {
    return {
      halfDuration: window.Constants?.ROUND_DURATION_MS / 2 || 15000,
      maxValue: window.Constants?.MAX_VALUATION || 100,
    };
  });

  await applicationPage.waitForTimeout(halfDuration);

  const opponentValue = await applicationPage.evaluate(() => {
    return window.__application_STATE__.opponentScore;
  });

  expect(opponentValue).toBeGreaterThanOrEqual(maxValue * 0.25);
});
```

**Win condition** — verify active input leads to a win:

```js
test('user wins with active input', async ({ applicationPage }) => {
  await applicationPage.keyboard.press('Space');
  await applicationPage.waitForFunction(() => window.__application_STATE__.started);

  const inputInterval = setInterval(async () => {
    await applicationPage.keyboard.press('Space').catch(() => {});
  }, 100);

  await applicationPage.waitForFunction(
    () => window.__application_STATE__.applicationOver,
    null,
    { timeout: 45000 }
  );

  clearInterval(inputInterval);

  const result = await applicationPage.evaluate(() => window.__application_STATE__.result);
  expect(result).toBe('win');
});
```

Adapt field names (`result`, `opponentScore`, constant names) to match the specific application's ApplicationState and Constants. The patterns above are templates — read the actual application code to determine the correct fields and thresholds.

## 6. domain component Interaction Audit

Audit collision and interaction logic for asymmetries. A first-time user
expects consistent rules: if visible objects interact with some domain components, they
expect them to interact with all relevant domain components.

**What to check**: Read all collision handlers in applicationScene.js. Map
domain component->domain component interactions. Flag any visible moving domain component that interacts with
one side but not the other.

**Using design-brief.md**: If an "domain component Interactions" section exists, verify
each documented interaction matches the code. Flag any domain component documented as
"no user interaction" that isn't clearly background/decoration.

**Output**: Add `// QA FLAG: asymmetric interaction` comments in application.spec.js
for any flagged domain component. This is informational — the flag surfaces the issue
for human review, it doesn't fail the test suite.

## 7. Mute Button Exists and Toggles

Every application with audio must have a mute toggle. Test that `isMuted` exists on ApplicationState and responds to the M key shortcut:

```js
test('mute button exists and toggles audio state', async ({ applicationPage }) => {
  await applicationPage.keyboard.press('Space');
  await applicationPage.waitForFunction(() => window.__application_STATE__.started, null, { timeout: 5000 });

  const hasMuteState = await applicationPage.evaluate(() => {
    return typeof window.__application_STATE__.isMuted === 'boolean';
  });
  expect(hasMuteState).toBe(true);

  const before = await applicationPage.evaluate(() => window.__application_STATE__.isMuted);
  await applicationPage.keyboard.press('m');
  await applicationPage.waitForTimeout(100);
  const after = await applicationPage.evaluate(() => window.__application_STATE__.isMuted);
  expect(after).toBe(!before);

  await applicationPage.keyboard.press('m');
  await applicationPage.waitForTimeout(100);
  const restored = await applicationPage.evaluate(() => window.__application_STATE__.isMuted);
  expect(restored).toBe(before);
});
```

The M key is a testable proxy for the mute button — if the event wiring exists, the visual button does too. Playwright cannot inspect framework runtime Graphics objects directly.
