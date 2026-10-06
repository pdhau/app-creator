---
name: application-design
description: Improve an application's information architecture, interaction design, visual hierarchy, responsive behavior, accessibility, and feedback states.
metadata:
  tags: UX, UI, accessibility, responsive, states
---

# Application Design

Audit the real workflow, then improve hierarchy, navigation, forms, loading, error, empty, success, and permission states. Preserve the existing design system and verify desktop, mobile, keyboard, contrast, and assistive technology behavior.---
name: design-application
description: Audit and improve the visual design, polish, and user experience of an existing application. Use when the user says "make my application look better", "improve the design", "add polish", "add juice", "add particles", "fix the UI", or "make it more visually appealing". Do NOT use for user workflow logic changes (use add-feature instead).
argument-hint: "[path-to-application]"
license: MIT
metadata:
  author: OpusapplicationLabs
  version: 1.3.0
  tags: [application, design, polish, ui, particles, visual]
---

## Performance Notes

- Take your time to do this thoroughly
- Quality is more important than speed
- Do not skip validation steps

# Design application

Run a UI/UX design pass on an existing application to improve visuals, atmosphere, and application feel. No design experience needed — this command analyzes your application and applies proven visual patterns.

## Instructions

Analyze the application at `$ARGUMENTS` (or the current directory if no path given).

First, load the application-designer skill to get the full design vocabulary and patterns.

### Step 1: Audit

- Read `package.json` to identify the framework runtime
- Read `src/core/Constants.js` for the current color palette and config
- Read all screen/view files to understand current visuals
- Read domain component files to see how application objects are drawn
- Read `src/core/EventBus.js` for existing events

### Step 2: Design Report

KPI each area 1-5 and present as a table:

| Area | KPI | Notes |
|------|-------|-------|
| Background & Atmosphere | | |
| Color Palette | | |
| Animations & Tweens | | |
| Particle Effects | | |
| Screen Transitions | | |
| Typography | | |
| application Feel / Juice | | |
| error/empty/success state | | |
| Expression Usage | | If personality domain models exist, KPI how reactively expressions change to application events. KPI 1 if expressions never change. |

Then list the top improvements ranked by visual impact, with a plain-English description of what each one does (e.g., "Add a sky gradient so the background looks like a real sky instead of a flat color").

### Step 3: Implement

Ask the user which improvements they want, or implement all if they say so. Follow the application-designer skill patterns:

1. All new values in `Constants.js`
2. Use EventBus for triggering effects
3. Don't alter user workflow (runtime behavior, scoring, controls, spawn timing)
4. Prefer procedural graphics
5. New files in proper directories

### Step 4: Verify

- Run `npm run build` to confirm no errors
- Summarize all changes made in plain English

## Example Usage

### Full design pass
```
/design-application examples/asteroid-dodge
```
Result: Audits visuals → scores Background 2/5, Particles 1/5, Typography 3/5 → adds sky gradient background, star parallax, explosion particles on asteroid destroy, screen shake on hit, smooth screen/view transitions. All values in Constants.js.

## Troubleshooting

### Visual changes cause performance drops
**Cause:** Too many particle emitters or gradient fills per frame.
**Fix:** Limit active particles (pool and reuse). Use cached gradient textures instead of recreating per-frame.

### Design changes break layout on different screen sizes
**Cause:** Hardcoded pixel positions instead of using PX scale factor.
**Fix:** All positions and sizes should use Constants.js PX-relative values.

## Next Step

Tell the user:

> Your application looks much better now! Next, run `/application-creator:add-audio` to add notification media background music and retro sound effects — all procedurally generated, no audio files needed.
>
> **Pipeline progress:** ~~/viral-application~~ → ~~/design-application~~ → `/add-audio` → `/qa-application` → `/review-application`
>
> *(This is the `/viral-application` one-shot pipeline — not the `/make-application` multi-session, milestone-driven workflow.)*
