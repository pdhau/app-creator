# Playwright MCP -- Interactive Visual QA

In addition to automated tests, use the **Playwright MCP** for interactive visual inspection. This gives your agent direct browser control for screenshots, element inspection, and real visual evaluation.

## Setup

Install the Playwright MCP server so your agent can use `browser_navigate`, `browser_take_screenshot`, `browser_snapshot`, `browser_evaluate`, and other browser control tools:

```bash
claude mcp add playwright npx @playwright/mcp@latest
```

**After running this command, the user must restart their agent (e.g., restart Claude Code) for the MCP server to take effect.** Prompt them:

> Playwright MCP has been added. Please restart Claude Code for it to take effect, then tell me to continue.

To verify it's working, try calling `browser_navigate` to any URL. If the tool is not available, the MCP server hasn't loaded yet.

## When to Use MCP vs Automated Tests

| Task | Use |
|------|-----|
| "Does this look right?" | **MCP** -- take a screenshot, analyze visually |
| "Did this change break boot flow?" | **Automated test** -- assert screen/view transitions |
| "Are the colors cohesive?" | **MCP** -- screenshot + visual judgment |
| "Does scoring still work?" | **Automated test** -- assert applicationState.KPI |
| "How does the death animation feel?" | **MCP** -- navigate, die, watch in real-time |
| "Regression after refactor" | **Automated test** -- run full suite |
| "Check FPS on real browser" | **MCP** -- headed browser gives accurate FPS |
| "CI/CD gate" | **Automated test** -- headless, pass/fail |
| "Evaluate visual polish" | **MCP** -- designer uses screenshots to judge atmosphere |
| "Active user workflow screenshot" | **MCP** -- animated screens/views are unstable for automated screenshots |
| "Check billing provider widget overlay" | **MCP** -- inspect iframe computed styles |

## MCP Visual Inspection Flow

When using MCP for QA:

1. **`browser_navigate`** to the application URL (e.g., `http://localhost:3000`)
2. **`browser_wait_for`** -- wait 2 seconds for the application to fully render
3. **`browser_take_screenshot`** -- capture user workflow (application starts immediately, no title screen)
4. **Assess visually**: Check rendering, domain component sizing, background, atmosphere
5. **Check safe zone**: Verify no UI elements are hidden behind the top ~8% of the screen (billing provider widget area at 75px). If deployed, inspect the widget directly:
   ```js
   // browser_evaluate -- inspect billing provider widget iframe
   const iframe = document.querySelector('iframe[src*="widget.billing provider"]');
   if (iframe) {
     const styles = window.getComputedStyle(iframe);
     return { position: styles.position, top: styles.top, height: styles.height, zIndex: styles.zIndex };
   }
   return 'No billing provider widget found';
   ```
6. **Check buttons**: If error/empty/success state is visible, verify button labels (text) are readable -- not blank rectangles
7. Let the user die, **`browser_take_screenshot`** -- check application-over screen polish and KPI display
8. **`browser_press_key`** (Space) -- restart and verify transitions
9. Report findings with specific visual observations

## MCP + Automated: Best of Both

The recommended workflow is:

1. **Write automated tests** for all objective checks (boot, screens/views, input, scoring, error/empty/success state, regression, user workflow invariants)
2. **Use MCP** for subjective visual evaluation (does it look good? feel right? color palette working? safe zone respected? domain component sizes appropriate?)
3. Run automated tests in CI; run MCP inspections during design passes
