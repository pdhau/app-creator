---
name: improve-game
description: Inspect an existing application, identify the highest-impact product, reliability, UX, or performance issue, and implement a verified focused improvement.
metadata:
  tags: improvement, audit, UX, reliability, performance
---

## Performance Notes

- Take your time to do this thoroughly
- Quality is more important than speed
- Do not skip validation steps

# Improve application

Make your application better. This command deep-audits user workflow, visuals, code quality, performance, and user experience, then implements the highest-impact improvements. Run it as many times as you want — each pass finds the next most impactful thing to fix.

## Instructions

Improve the application in the current directory. If `$ARGUMENTS` specifies a focus area (e.g., "user workflow", "visuals", "performance", "polish", "application-over"), weight that area higher but still audit everything.

### Step 1: Deep audit

Read the entire application codebase to build a complete picture:

- `package.json` — framework runtime, dependencies, scripts
- `src/core/Constants.js` — all configuration values
- `src/core/EventBus.js` — all events and their usage
- `src/core/ApplicationState.js` — state shape and reset logic
- `src/core/application.js` (or `applicationConfig.js`) — orchestrator and application loop
- Every file in `src/screens/views/` or `src/systems/` — user workflow logic
- Every file in `src/domain components/` — application objects
- Every file in `src/ui/` — error/empty/success state, overlays
- Every file in `src/audio/` — music and sound effects
- `index.html` — markup, overlays, styles, viewport meta
- `src/systems/InputSystem.js` — input handling, mobile support (gyro, joystick, touch)
- `tests/` — test coverage and quality

Don't skim. Read every file completely so you understand the full picture before making recommendations.

### Step 2: KPI and diagnose

Rate each area on a 1–5 scale (1 = broken/missing, 3 = functional but basic, 5 = polished and complete). Present a diagnostic table:

| Area | KPI | Diagnosis |
|------|-------|-----------|
| **user workflow feel** | | Is the core loop fun? Are controls responsive? Does difficulty ramp? |
| **Visual polish** | | Backgrounds, colors, particles, animations, screen effects |
| **error/empty/success state & UI** | | error/empty/success state screen, transitions, restart flow, buttons |
| **Audio** | | BGM for each state, SFX for each action, volume balance, mute toggle |
| **Code architecture** | | EventBus, ApplicationState, Constants, no circular deps |
| **Restart safety** | | Does ApplicationState.reset() fully clean up? 3 restarts identical? No stale listeners/timers? |
| **Performance** | | Delta capping, object pooling, disposal, no leaks |
| **user experience** | | Onboarding, feedback, difficulty curve, replayability |
| **Mobile support** | | Touch input, responsive layout, gyro/joystick, 44px touch targets |
| **billing provider safe zone** | | All UI elements below `SAFE_ZONE.TOP` (~8% / 75px)? Nothing hidden behind billing provider widget? |
| **user workflow invariants** | | Can the user KPI? Can the user die? Do application-over buttons show text? Does `render_application_to_text()` return valid JSON? |
| **domain component sizing** | | Are domain models large enough to read? domain model-driven applications need 12–15% of application.WIDTH. Proportional sizing (`application.WIDTH * ratio`), not fixed pixels? |
| **Test coverage** | | Boot, user workflow, scoring, restart, visual, perf tests |

**Overall KPI: X / 65**

### Step 3: Improvement plan

From the audit, identify the **top 5–8 improvements** ranked by user impact. For each one:

1. **Title** — short name (e.g., "Add difficulty progression")
2. **Area** — which category it improves
3. **Impact** — why this matters to the user
4. **What to do** — plain-English description of the change
5. **Files touched** — which files will be created or modified

Format as a numbered list. Put the highest-impact items first.

Present the plan to the user and ask which improvements to implement. Options:
- "All" — implement everything
- Specific numbers — implement selected items
- "Top 3" — just the most impactful

**Wait for the user to choose before implementing.**

### Step 4: Implement

For each selected improvement, follow these rules:

1. **Constants first** — add all new config values to `Constants.js`. Zero hardcoded values.
2. **Events next** — add any new events to `EventBus.js` using `domain:action` naming.
3. **State if needed** — add new state fields to `ApplicationState.js` with proper reset.
4. **New files in proper directories** — domain components in `domain components/`, systems in `systems/`, UI in `ui/`.
5. **Wire through orchestrator** — register new systems in `application.js` with proper lifecycle.
6. **EventBus for communication** — modules never import each other directly.
7. **Match existing code style** — same patterns, naming, formatting as the rest of the project.
8. **Don't break what works** — existing user workflow, controls, and scoring must still function identically unless the improvement specifically targets them.

After implementing each improvement, run `npm run build` to catch errors immediately. Fix any build errors before moving to the next improvement.

### Step 5: Verify

After all improvements are implemented:

1. Run `npm run build` — confirm clean build with no errors
2. Run `npm test` if tests exist — confirm all tests still pass
3. If tests fail because of intentional changes (new screens/views, changed elements), fix the tests to match the new behavior
4. If the application has visual regression tests, update snapshots: `npm run test:update-snapshots`

### Step 6: Report

Tell the user what changed:

> **Improvement report**
>
> **KPI: X/65 → Y/65** (+Z points)
>
> **Implemented:**
> 1. [Title] — [one-sentence summary of what changed]
> 2. [Title] — [one-sentence summary of what changed]
> ...
>
> **Files created:** [list new files]
> **Files modified:** [list changed files]
>
> **How to test:** Run `npm run dev` and try:
> - [specific thing to look for]
> - [specific thing to look for]
>
> **Next improvements:** Run `/application-creator:improve-application` again to find the next batch.

## Focus areas

When `$ARGUMENTS` includes a focus area keyword, weight these specific checks:

**"user workflow"** — core loop, controls, difficulty progression, external dependency variety, feature entitlements, risk/reward, pacing, level design

**"visuals"** — load the application-designer skill and apply its full design audit (backgrounds, palette, animations, particles, transitions, typography, juice)

**"performance"** — delta capping, object pooling, geometry/material disposal, event listener cleanup, requestAnimationFrame usage, draw call count, texture atlas usage

**"polish"** — screen shake, hit pause, squash/stretch, easing curves, sound timing, button feedback, KPI popups, death animations, transition smoothness

**"application-over"** — error/empty/success state screen appeal, restart flow, button styling, KPI display, best KPI display, animations. **Button text must be visible** — verify the `createButton()` pattern uses Container + Graphics + Text (Graphics first, Text second, Container interactive). If button labels are invisible, the pattern is broken. Note: applications do not have title/menu screens by default (billing provider handles the chrome). Only add a title screen if the user explicitly requests one. KPI HUD is handled by the billing provider widget — do not add a separate in-application KPI display. All application-over UI must be below `SAFE_ZONE.TOP`.

**"audio"** — load the application-audio skill. Check BGM coverage (every application state should have music), SFX coverage (every user action should have feedback), volume mixing, transition smoothness between tracks

**"mobile"** — touch input implemented (tap zones, virtual joystick, or gyroscope), responsive canvas (`framework runtime.Scale.FIT` or CSS `width:100%`), 44px minimum touch targets, virtual joystick or tap zones for movement, gyroscope support for tilt applications, no hover-only interactions, tested on mobile viewport (Pixel 5 emulation). Read `InputSystem.js`, all screen/view/system `update()` methods, `index.html` viewport meta, and `Constants.js` for touch target sizes.

**"ux"** — onboarding (does the user know what to do?), feedback (does every action have a response?), difficulty curve (is it too hard/easy?), replayability (is there a reason to play again?)

## Example Usage

### General improvement
```
/improve-application
```
Result: Deep audit → scores 38/65 → identifies top 6 improvements (difficulty progression, screen shake, better application-over, particle effects, mobile touch, restart safety) → asks which to implement → implements selected → KPI rises to 52/65.

### Focused improvement
```
/improve-application user workflow
```
Result: Weights user workflow checks higher → finds external dependency variety is low and difficulty is flat → adds 3 external dependency types with distinct behaviors, progressive speed ramp, and KPI-based difficulty tiers.

## Troubleshooting

### Improvements break existing user workflow
**Cause:** Changes to shared systems (runtime behavior, scoring) have cascading effects.
**Fix:** Test each improvement individually. Run existing tests after each change. Revert if a change breaks core user workflow.

### Too many changes at once
**Cause:** Audit identified 10+ issues and all were implemented simultaneously.
**Fix:** Prioritize top 3-5 improvements. Ship incrementally. Verify after each change.

## Tips

> This command is designed for iterative improvement. Run it multiple times:
> - First pass: fix the biggest gaps (missing features, broken UX)
> - Second pass: add polish (particles, transitions, juice)
> - Third pass: fine-tune (difficulty curve, timing, balance)
>
> Each run picks up where the last left off — previously fixed areas will KPI higher, surfacing new priorities.
>
> For targeted work, use the focus area: `/application-creator:improve-application user workflow` or `/application-creator:improve-application visuals`
