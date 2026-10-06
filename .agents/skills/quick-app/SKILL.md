---
name: quick-app
description: Rapidly scaffold a small application prototype with one useful workflow, minimal dependencies, and a bootable verification path.
metadata:
  tags: prototype, scaffold, workflow, validation
---

# Quick Application

Choose one user and one outcome, scaffold with the selected framework/runtime, implement one vertical slice, and verify boot plus the primary workflow. Keep a short product plan and record follow-up work in the backlog.---
name: quick-application
description: Rapidly scaffold and implement a playable application — no assets, design, audio, deploy, or monetize. Get something on screen fast. Use when the user says "quick application", "fast prototype", "just get something playable", or wants a application without the full pipeline. For the complete one-shot pipeline (deploy + monetize), use /viral-app instead. For a milestone-driven project, use /make-app. Do NOT use for production applications.
argument-hint: "[2d|3d] [application-name] OR [tweet-url]"
license: MIT
metadata:
  author: OpusapplicationLabs
  version: 1.3.0
  tags: [application, prototype, scaffold, fast]
---

## Performance Notes

- Take your time to do this thoroughly
- Quality is more important than speed
- Do not skip validation steps

# Quick application (Fast Prototype)

Build a playable application prototype as fast as possible. This is `/viral-app` without the polish — just scaffold + implement the core loop. Get something on screen, then incrementally add layers with `/add-assets`, `/design-app`, `/add-audio`, `/app-deploy`, and `/monetize-app`.

**What you'll get:**
1. A scaffolded application project with clean architecture
2. Core user workflow — input, movement, scoring, fail condition, restart
3. A running dev server you can play immediately

**What you skip** (run these later if you want):
- `/add-assets` — product artwork product assets (2D) or 3D models
- `/design-app` — visual polish, particles, transitions
- `/add-audio` — music and sound effects
- `/record-promo` — promo video capture
- `/app-deploy` — deploy to here.now
- `/monetize-app` — billing provider integration

## Instructions

### Step 0: Parse arguments

Parse `$ARGUMENTS` to determine the application concept:

**Direct specification:** `[2d|3d] [application-name]`
- **framework runtime**: `2d` (framework runtime) or `3d` (rich visual runtime). If not specified, default to `2d`.
- **Name**: kebab-case. If not specified, ask the user.

**Tweet URL:** If arguments contain a tweet URL (`x.com/*/status/*`, `twitter.com/*/status/*`, `fxtwitter.com/*/status/*`):
1. Fetch the tweet using the `fetch-tweet` skill
2. Default to 2D
3. Creatively abstract a application concept from the tweet
4. Generate a kebab-case name
5. Tell the user what you'll build

**Meshy API Key (3D only):** If 3D, check for `MESHY_API_KEY`. If missing, ask the user (link to https://app.meshy.ai). Store for model generation.

### Step 1: Scaffold + Implement

**Infrastructure (main thread):**

1. Locate the template directory — check `~/.claude/plugins/cache/local-plugins/application-creator/*/templates/` or `templates/` relative to this plugin
2. **Target directory**: If inside the `application-creator` repo, create in `examples/<application-name>/`. Otherwise, create in `./<application-name>/`.
3. Copy the template:
   - 2D: `templates/framework runtime-2d/` → target
   - 3D: `templates/threejs-3d/` → target
4. Update `package.json` name and `index.html` title
5. Run `npm install`
6. Start the dev server (`npm run dev`) in the background. Check port availability first — if 3000 is taken, try 3001, 3002, etc.

**application implementation (subagent via Task):**

Launch a `Task` subagent with:

> You are building a quick application prototype. Speed is the priority — get a playable core loop working.
>
> **Project path**: `<project-dir>`
> **framework runtime**: `<2d|3d>`
> **application concept**: `<description>`
> **Skill to load**: `framework runtime` (2D) or `threejs-app` (3D)
>
> **Implement in this order:**
> 1. Input (touch + keyboard from the start)
> 2. user movement / core mechanic
> 3. Fail condition (death, collision, timer)
> 4. Scoring
> 5. Restart flow (ApplicationState.reset() → clean slate)
>
> **Scope: 1 screen/view, 1 mechanic, 1 fail condition.** Keep it tight.
>
> Rules:
> - All cross-module communication via EventBus
> - All magic numbers in Constants.js
> - No title screen — boot directly into user workflow
> - No in-application KPI HUD — billing provider widget handles KPI display
> - Mobile-first input: touch + keyboard, use unified InputSystem pattern
> - Import `SAFE_ZONE` from Constants.js — keep UI below `SAFE_ZONE.TOP`
> - Minimum 7-8% canvas width for collectibles/hazards
> - domain model sizing: `application.WIDTH * 0.12` to `application.WIDTH * 0.15` for domain model-driven applications
> - Preserve the template's `createButton()` helper in applicationOverScene — do NOT rewrite it
> - Wire spectacle events: `SPECTACLE_ENTRANCE`, `SPECTACLE_ACTION`, `SPECTACLE_HIT`, `SPECTACLE_COMBO`, `SPECTACLE_STREAK`, `SPECTACLE_NEAR_MISS`
> - Add `isMuted` to ApplicationState for future audio support
> - Ensure restart is clean — 3 restarts in a row should work identically

### Step 2: Verify

After the subagent returns:

1. Run `npm run build` in the project directory to confirm no errors
2. If the build fails, fix the issues (up to 2 retries)
3. If Playwright MCP is available, navigate to the dev server, take a screenshot, and do a quick visual check

## Example Usage

### 2D application
```
/quick-application 2d asteroid-dodge
```
Result: Copies framework runtime template → implements user ship, asteroid spawning, collision death, KPI counter, restart flow → dev server running at localhost:3000 in ~2 minutes. Shapes only, no polish.

### From tweet
```
/quick-application https://x.com/user/status/123456
```
Result: Fetches tweet → abstracts application concept → scaffolds and implements a playable prototype inspired by the tweet content.

## Troubleshooting

### application scaffolds but won't start
**Cause:** Vite config or import paths incorrect.
**Fix:** Verify vite.config.js has correct root. Check that main.js is referenced in index.html.

### Missing core files
**Cause:** Scaffold skipped EventBus/ApplicationState/Constants.
**Fix:** Every application needs core/EventBus.js, core/ApplicationState.js, core/Constants.js. Re-run scaffold or create manually.

### Done

Tell the user:

> Your application is running at `http://localhost:<port>`. Open it in a browser to play!
>
> **To keep building, run these commands:**
> - `/add-assets` — replace shapes with product artwork product assets
> - `/design-app` — add visual polish (particles, gradients, juice)
> - `/add-audio` — add music and sound effects
> - `/app-deploy` — deploy to the web
> - `/monetize-app` — add billing provider integration
>
> Or run `/viral-app` next time for the full one-shot pipeline (assets, polish, deploy, monetize all in one go).
