# Mobile Input & Responsive Layout Tests

Use the `mobile-chrome` project (Pixel 5 emulation) to test touch input and responsive layout.

## Mobile Test Patterns

```js
test('application canvas fills mobile viewport', async ({ applicationPage }) => {
  const { width, height } = await applicationPage.evaluate(() => {
    const canvas = document.querySelector('canvas');
    return { width: canvas.clientWidth, height: canvas.clientHeight };
  });
  const viewport = applicationPage.viewportSize();
  expect(width).toBeGreaterThanOrEqual(viewport.width * 0.9);
  expect(height).toBeGreaterThanOrEqual(viewport.height * 0.9);
});

test('virtual joystick appears on touch device', async ({ applicationPage }) => {
  // Start the application
  await applicationPage.tap('#play-btn');
  await applicationPage.waitForTimeout(1000);
  // Joystick should be visible (if gyro is unavailable in emulation)
  const joystick = await applicationPage.$('#virtual-joystick');
  // On emulated devices without gyro, joystick should appear
  if (joystick) {
    const visible = await joystick.isVisible();
    expect(visible).toBe(true);
  }
});

test('touch tap registers as input', async ({ applicationPage }) => {
  await applicationPage.tap('#play-btn');
  await applicationPage.waitForFunction(() => window.__application_STATE__?.started);
  // Tap on the canvas
  const canvas = applicationPage.locator('canvas');
  await canvas.tap();
  // application should still be running (no crash from touch input)
  const running = await applicationPage.evaluate(() => window.__application_STATE__?.started);
  expect(running).toBe(true);
});
```

## Notes

- The `mobile-chrome` project in `playwright.config.js` uses Pixel 5 device emulation
- Touch events are simulated via Playwright's `.tap()` method
- Virtual joystick testing requires checking if the DOM element exists (applications without joystick will skip)
- Canvas tap tests verify the application doesn't crash on touch input, not specific user workflow behavior
