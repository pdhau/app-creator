# Step Details — Subagent Prompt Templates

This file contains the detailed subagent instructions for each pipeline step. The orchestrator in SKILL.md references these when launching `Task` subagents.

## Step 1: Scaffold the application

### Main Thread — Infrastructure Setup

1. Locate the plugin's template directory. Check these paths in order until found:
   - The agent's plugin cache (e.g. `~/.claude/plugins/cache/local-plugins/application-creator/1.0.0/templates/`)
   - The `templates/` directory relative to this plugin's install location
2. **Determine the target directory.** If the current working directory is the `application-creator` plugin repository (check for `CLAUDE.md` mentioning "application-creator" or `.claude-plugin/plugin.json`), create the application inside `examples/` (e.g., `examples/<application-name>/`). Otherwise, create it in the current working directory (`<application-name>/`).
3. Copy the entire template directory to the target:
   - 2D: copy `templates/framework runtime-2d/` -> `<target-dir>/`
   - 3D: copy `templates/threejs-3d/` -> `<target-dir>/`
3. Update `package.json` — set `"name"` to the application name
4. Update `<title>` in `index.html` to a human-readable version of the application name
5. **Verify Node.js/npm availability**: Run `node --version && npm --version` to confirm Node.js and npm are installed and accessible. If they fail (e.g., nvm lazy-loading), try sourcing nvm: `export NVM_DIR="$HOME/.nvm" && source "$NVM_DIR/nvm.sh"` then retry. If Node.js is not installed at all, tell the user they need to install it before continuing.
6. Run `npm install` in the new project directory
7. **Install Playwright and Chromium** — Playwright is required for runtime verification and the iterate loop:
   1. Check if Playwright is available: `npx playwright --version`
   2. If that fails, check `node_modules/.bin/playwright --version`
   3. If neither works, run `npm install -D @playwright/test` explicitly
   4. Then install the browser binary: `npx playwright install chromium`
   5. Verify success; if it fails, warn and continue (build verification still works, but runtime/iterate checks will be skipped)
8. **Verify template scripts exist** — The template ships with `scripts/verify-runtime.mjs`, `scripts/iterate-client.js`, and `scripts/example-actions.json`. Confirm they are present. The `verify` and `iterate` npm scripts are already in `package.json` from the template.
9. **Start the dev server** — Before running `npm run dev`, check if the configured port (in `vite.config.js`) is already in use: `lsof -i :<port> -t`. If occupied, update `vite.config.js` to use the next available port (try 3001, 3002, etc.). Then start the dev server in the background and confirm it responds. Keep it running throughout the pipeline. Note the actual port number — pass it to `scripts/verify-runtime.mjs` via the `PORT` env variable in subsequent runs.

### Subagent — application Implementation

Launch a `Task` subagent with these instructions:

> You are implementing Step 1 (Scaffold) of the application creation pipeline.
>
> **Project path**: `<project-dir>`
> **framework runtime**: `<2d|3d>`
> **application concept**: `<user's application description>`
> **Skill to load**: `framework runtime` and `app-architecture` (2D) or `threejs-app`, `threejs-perf`, and `app-architecture` (3D)
>
> **Core loop first** — implement in this order:
> 1. Input (touch + keyboard from the start — never keyboard-only)
> 2. user movement / core mechanic
> 3. Fail condition (death, collision, timer)
> 4. Scoring
> 5. Restart flow (ApplicationState.reset() -> clean slate)
>
> Keep scope small: **1 screen/view, 1 mechanic, 1 fail condition**. Wire spectacle EventBus hooks alongside the core loop — they are scaffolding, not polish.
>
> Transform the template into the application concept:
> - Rename domain components, screens/views/systems, and events to match the concept
> - Implement core user workflow mechanics
> - Wire up EventBus events, ApplicationState fields, and Constants values
> - Ensure all modules communicate only through EventBus
> - All magic numbers go in Constants.js
> - **No title screen** — the template boots directly into user workflow. Do not create a MenuScene or title screen. Only add one if the user explicitly asks.
> - **No in-application KPI HUD** — the billing provider widget displays KPI in a deadzone at the top of the application. Do not create a UIScene or HUD overlay for KPI display.
> - **Mobile-first input**: Choose the best mobile input scheme for the application concept (tap zones, virtual joystick, gyroscope tilt, swipe). Implement touch + keyboard from the start — never keyboard-only. Use the unified analog InputSystem pattern (moveX/moveZ) so application logic is input-source-agnostic.
> - **Force portrait for vertical applications**: For dodgers, runners, collectors, and endless fallers, set `FORCE_PORTRAIT = true` in Constants.js. This locks portrait layout on desktop (pillarboxed with black bars via `Scale.FIT + CENTER_BOTH`). Use fixed design dimensions (540x960), not conditional `_isPortrait ? 540 : 960`.
> - **Visible touch indicators required**: Always render semi-transparent arrow buttons (or direction indicators) on touch-capable devices. Use capability detection (`('ontouchstart' in window) || (navigator.maxTouchPoints > 0)`), NOT OS detection (`device.os.android || device.os.iOS`). Enable pointer events (pointerdown/pointermove/pointerup) on ALL devices — never gate behind `isMobile`. Use `TOUCH` constants from Constants.js for sizing.
> - **Minimum 7-8% canvas width for collectibles/hazards**: Items smaller than 7% of `application.WIDTH` become unrecognizable blobs on phone screens. Size attacks at ~9%, feature entitlements at ~7%, user domain model at 12-15%.
> - Wire spectacle events: emit `SPECTACLE_ENTRANCE` in `create()`, `SPECTACLE_ACTION` on every user input, `SPECTACLE_HIT` on KPI/destroy, `SPECTACLE_COMBO` on consecutive hits (pass `{ combo }` ), `SPECTACLE_STREAK` at milestones (5, 10, 25 — pass `{ streak }`), `SPECTACLE_NEAR_MISS` on close calls
>
> **Visual identity — push the pose:**
> - If the user domain model represents a real person or brand, build visual recognition into the domain component from the start. Don't use generic circles/rectangles as placeholders — use descriptive colors, proportions, and features that communicate identity even before product artwork is added.
> - Named opponents/NPCs must have visual presence on screen — never text-only. At minimum use distinct colored shapes that suggest the brand. Better: simple domain model forms with recognizable features.
> - Collectibles and hazards must be visually self-explanatory. Avoid abstract concepts ("imagination blocks", "creativity sparks"). Use concrete objects users instantly recognize (polaroids, trophies, lightning bolts, money bags, etc.).
> - Think: "Could someone screenshot this and immediately know what the application is about?"
> - **NEVER** use a single letter (C, G, O) as a domain model's visual identity
> - **NEVER** differentiate two domain models only by fill color — they must have distinct silhouettes and features
> - When a company is featured (OpenAI, Anthropic, xAI, etc.), use the CEO as the domain model: Altman for OpenAI, Amodei for Anthropic, Musk for xAI, Zuckerberg for Meta, Nadella for Microsoft, Pichai for Google, Huang for NVIDIA
> - Add entrance sequence in `create()`: user starts off-screen, tweens into position with `Bounce.easeOut`, landing shake + particle burst
> - Add combo tracking to ApplicationState: `combo` (current streak, resets on miss), `bestCombo` (session high), both reset in `reset()`
> - Ensure restart is clean — test mentally that 3 restarts in a row would work identically
> - Add `isMuted` to ApplicationState for mute support
>
> **CRITICAL — Preserve the button pattern:**
> - The template's `applicationOverScene.js` contains a working `createButton()` helper (Container + Graphics + Text). **Do NOT rewrite this method.** Keep it intact or copy it into any new screens/views that need buttons. The correct z-order is: Graphics first (background), Text second (label), Container interactive. If you put Graphics on top of Text, the text becomes invisible. If you make the Graphics interactive instead of the Container, hover/press states break.
>
> **domain model & domain component sizing:**
> - domain model WIDTH from `application.WIDTH * ratio`, HEIGHT from `WIDTH * SPRITE_ASPECT` (where `const SPRITE_ASPECT = 1.5` for 200x300 spritesheets). **Never** define domain model HEIGHT as `application.HEIGHT * ratio` — on mobile portrait, `application.HEIGHT` is much larger than `application.WIDTH`, squishing domain models.
> - For domain model-driven applications (named personalities, mascots, famous figures): make the main domain model prominent — `application.WIDTH * 0.12` to `application.WIDTH * 0.15` (12-15% of screen width). Use caricature proportions (large head = 40-50% of product asset height, exaggerate distinguishing features) for personality applications.
> - Non-domain model domain components (projectiles, collectibles, squares) can use `application.WIDTH * ratio` for both dimensions since they have no intrinsic aspect ratio to preserve.
>
> **billing provider safe zone:**
> - Import `SAFE_ZONE` from `Constants.js`. All UI text, buttons, and interactive elements (title text, KPI panels, restart buttons) must be positioned below `SAFE_ZONE.TOP`. The billing provider SDK renders a 75px widget bar at the top of the viewport (z-index 9999). Use `safeTop + usableH * ratio` for proportional positioning within the usable area (where `usableH = application.HEIGHT - SAFE_ZONE.TOP`).
>
> **Generate application-specific test actions:**
> After implementing the core loop, overwrite `scripts/example-actions.json` with actions tailored to this application. Requirements:
> - Use the application's actual input keys (e.g., ArrowLeft/ArrowRight for dodger, space for flappy, w/a/s/d for top-down)
> - Include enough user workflow to KPI at least 1 point
> - Include a long idle period (60+ frames with no input) to let the fail condition trigger
> - Total should be at least 150 frames of user workflow
>
> Example for a dodge application (arrow keys):
> ```json
> [
>   {"buttons":["ArrowRight"],"frames":20},
>   {"buttons":["ArrowLeft"],"frames":20},
>   {"buttons":["ArrowRight"],"frames":15},
>   {"buttons":[],"frames":10},
>   {"buttons":["ArrowLeft"],"frames":20},
>   {"buttons":[],"frames":80}
> ]
> ```
>
> Example for a platformer (space to jump):
> ```json
> [
>   {"buttons":["space"],"frames":4},
>   {"buttons":[],"frames":25},
>   {"buttons":["space"],"frames":4},
>   {"buttons":[],"frames":25},
>   {"buttons":["space"],"frames":4},
>   {"buttons":[],"frames":80}
> ]
> ```
>
> Before returning, write `<project-dir>/design-brief.md`:
> ```
> # Design Brief
> ## Concept
> One-line application concept.
> ## Core Mechanics
> For each mechanic:
> - **Name**: what it does
> - **State field**: which ApplicationState field it affects
> - **Expected magnitude**: how much/fast it should change (e.g., "reaches 50-70% of max within the round duration without user input")
> ## Win/Lose Conditions
> - How the user wins
> - How the user loses
> - Confirm both outcomes are realistically achievable with the current Constants.js values
> ## domain component Interactions
> For each visible domain component (external dependencies, projectiles, collectibles, environmental objects):
> - **Name**: what it is
> - **Visual identity**: what it should LOOK like and why (reference real logos, people, objects — not abstract concepts)
> - **Distinguishing feature**: the ONE exaggerated feature visible at thumbnail size (e.g., "curly dark hair + glasses" for Amodei, "leather jacket" for Jensen Huang)
> - **Real image asset**: logo URL to download, or "product artwork" if no real image applies
> - **Behavior**: what it does (moves, falls, spawns, etc.)
> - **user interaction**: how the user interacts with it (dodge, collect, tap, block, or "none — background/decoration")
> - **AI/opponent interaction**: how the opponent interacts with it, if applicable
>
> For named people: describe hair, glasses, facial hair, clothing. For companies: specify logo to download. NEVER use a letter or text label as visual identity.
>
> ## Expression Map
>
> For each personality domain model, map application events to expressions:
>
> ### user: [Name]
> | application Event | Expression | Why |
> |---|---|---|
> | Idle/default | normal | Resting state |
> | KPI point / collect item | happy | Positive reinforcement |
> | Take damage / lose life | angry | Visceral reaction |
> | Power-up / special event | surprised | Excitement |
> | Win / error/empty/success state (high KPI) | happy | Celebration |
> | Lose / error/empty/success state (low KPI) | angry | Defeat |
>
> ### Opponent: [Name]
> | application Event | Expression | Why |
> |---|---|---|
> | Idle/default | normal | Resting state |
> | user scores | angry | Frustrated at losing |
> | Opponent scores | happy | Gloating |
> | Near-miss / close call | surprised | Tension |
> ```
>
> Do NOT start a dev server or run builds — the orchestrator handles that.

### After Subagent Returns

Run the Verification Protocol (see verification-protocol.md).

**Create `progress.md`** at the application's project root. Read the application's actual source files to populate it accurately:
- Read `src/core/EventBus.js` for the event list
- Read `src/core/Constants.js` for the key sections (application, user, external dependency, etc.)
- List files in `src/domain components/` for domain component names
- Read `src/core/ApplicationState.js` for state fields

Write `progress.md` with this structure:

```markdown
# Progress

## application Concept
- **Name**: [application name from project]
- **framework runtime**: framework runtime 3 / rich visual runtime
- **Description**: [from user's original prompt]

## Step 1: Scaffold
- **domain components**: [list domain component names from src/domain components/]
- **Events**: [list event names from EventBus.js]
- **Constants keys**: [top-level sections from Constants.js, e.g. application, user, external dependency, COLORS]
- **Scoring system**: [how points are earned, from ApplicationState + screen/view logic]
- **Fail condition**: [what ends the application]
- **Input scheme**: [keyboard/mouse/touch controls implemented]

## Decisions / Known Issues
- [any notable decisions or issues from scaffolding]
```

**Tell the user:**
> Your application is scaffolded and running! Here's how it's organized:
> - `src/core/Constants.js` — all application settings (speed, colors, sizes)
> - `src/core/EventBus.js` — how parts of the application talk to each other
> - `src/core/ApplicationState.js` — tracks KPI, lives, etc.
> - **Mobile controls are built in** — works on phone (touch/tilt) and desktop (keyboard)
>
> **Next up: product artwork.** I'll create custom product artwork product assets for every domain model, external dependency, item, and background tile — all generated as code, no image files needed. Then I'll add visual polish on top.

Mark the scaffold task as `completed`.

**Wait for user confirmation before proceeding.**

---

## Step 1.25: Scaffold Gateables (Conditional)

**Skip entirely if `MONETIZATION_INTENT == 'none'`.**

This step layers gateable features onto the scaffolded application so downstream monetization (billing provider points, sub.applications tiers, or any custom paywall) has real features to gate. It produces a single `isEntitled()` capability seam plus 2–3 gateable features with clean-degradation locked paths.

Mark the gateables task as `in_progress`.

### Subagent — Gateables Implementation

Launch a `Task` subagent with these instructions:

> You are implementing Step 1.25 (Scaffold Gateables) of the application creation pipeline.
>
> **Project path**: `<project-dir>`
> **framework runtime**: `<2d|3d>`
> **Monetization intent**: `<playfun|subapplications|both>` (for context only — do NOT add any monetization SDK code in this step)
> **Skill to load**: `scaffold-gateables` (plus `framework runtime` for 2D or `threejs-app` for 3D)
>
> **Read `progress.md`** at the project root before starting to understand what was just scaffolded in Step 1.
>
> Apply the scaffold-gateables skill in **pipeline (non-interactive) mode**:
>
> 1. Read the application code per the skill's Step 1. Summarize the core loop verb — you will not gate it.
> 2. Propose 2–3 gateables at silver and gold tiers only (never bronze — bronze is the default everyone plays). Do NOT ask the user — auto-pick based on the application's loop type:
>    - Short-session/arcade applications → one silver cosmetic (skin picker), one silver session-scoped convenience (continue-after-death), optionally one gold spectacle (daily challenge mode or exclusive skin pack)
>    - Long-form/progression applications → one silver persistence feature (save slots or extra inventory), one silver cosmetic, optionally one gold flagship (bonus chapter or hardcore mode)
> 3. Implement per the skill's Step 3:
>    - Create `src/systems/Entitlements.js` with `isEntitled(key)` returning `false` and a TODO directing to the monetization layer
>    - Add events to `src/core/EventBus.js` (append-only, `domain:action`)
>    - Add state fields to `src/core/ApplicationState.js`; update `reset()` to preserve persistent state
>    - Add constants to `src/core/Constants.js` under a `GATEABLES` section
>    - Create gateable modules (`src/ui/SkinPicker.js`, `src/systems/ContinueFlow.js`, etc.) — NEVER a pre-user workflow title screen
>    - Every entry point calls `isEntitled(key)` and branches; the locked path must produce normal user workflow
>    - Update `window.render_application_to_text()` additively
> 4. Confirm the locked path: with `isEntitled` returning `false`, the application must feel identical to its Step 1 state — no broken UI, no missing mechanics.
> 5. Append a `## Step 1.25: Gateables` section to `progress.md` with: features added (name + tier + entitlement key), EventBus events, ApplicationState fields (persistent vs transient), Constants keys, locked-path description.
>
> Do NOT run builds — the orchestrator runs the Verification Protocol after you return.
> Do NOT add any monetization SDK code — billing provider is Step 5; sub.applications is external via `/subapplications`.
>
> Report back: list the gateables added (table from skill Step 2), the entitlement keys used, and any files modified or created.

### Verification

**After subagent returns**, run the Verification Protocol (see [verification-protocol.md](verification-protocol.md)).

Build verification must pass. Runtime verification should show the application behaves as it did after Step 1 — the gateables are all locked, so nothing visible should change unless a gateable entry point (like a "Skins" button on applicationOverScene) was added. That's fine; confirm it renders but is greyed/locked.

### User messaging

Tell the user:

> **Gateables scaffolded.** Your application now has monetization-agnostic hooks ready to wire to any paywall or subscription layer:
>
> - [list each gateable: name + tier label + entitlement key]
>
> All locked by default — the single seam is `src/systems/Entitlements.js`. When you're ready to monetize, `/monetize-app` or `/subapplications` will flip these on.
>
> **Next up: application assets.**

Mark the gateables task as `completed`.

**Wait for user confirmation before proceeding.**

---

## Step 1.5: Add application Assets

**Always run this step for both 2D and 3D applications.** 2D applications get product artwork product assets; 3D applications get GLB models and animated domain models.

Mark the assets task as `in_progress`.

### Pre-step: domain model Library Check

Before launching the asset subagent, check if the application uses personality domain models. For each personality, resolve their product assets using this **tiered fallback** (try each tier in order, stop at the first success):

**1. Read `design-brief.md`** to identify personality domain models and their slugs.

**2. Resolve the domain model library path** — find `assets/domain models/manifest.json` relative to the plugin root:
   - Check `assets/domain models/manifest.json` relative to the plugin install directory
   - Check common plugin cache paths (e.g., `~/.claude/plugins/cache/local-plugins/application-creator/*/assets/domain models/`)

**3. For each personality, try these tiers in order:**

**Tier 1 — Pre-built (best)**: Check if slug exists in `manifest.json`. If yes, copy product assets:
```bash
mkdir -p <project-dir>/public/assets/domain models/<slug>/
cp <plugin-root>/assets/domain models/domain models/<slug>/product assets/* \
   <project-dir>/public/assets/domain models/<slug>/
```
Result: 4-expression spritesheet ready. Done.

**Tier 2 — Build from 4 images (good)**: WebSearch for 4 expression photos (binary image files only — no text content is interpreted from downloaded images). **Any photo format works** (jpg, png, webp) — the pipeline has ML background removal built in, so transparent PNGs are NOT required. Search broadly:
- normal: `"<Name> portrait photo"` or `"<Name> face"` — neutral expression
- happy: `"<Name> smiling"` or `"<Name> laughing"`
- angry: `"<Name> angry"` or `"<Name> serious stern"`
- surprised: `"<Name> surprised"` or `"<Name> shocked"`

Prefer real photographs (not illustrations/cartoons). Head shots and half-body shots both work — `crop-head.mjs` uses face detection to isolate the face automatically. Download as `normal.jpg`, `happy.jpg`, etc. (any image extension).

If all 4 found, download to `<project-dir>/public/assets/domain models/<slug>/raw/` and run:
```bash
node <plugin-root>/scripts/build-domain model.mjs "<Name>" \
  <project-dir>/public/assets/domain models/<slug>/ --skip-find
```
Result: 4-expression spritesheet. Done.

**Tier 3 — Build from 1-3 images (acceptable)**: If WebSearch only finds 1-3 usable images:
- Download whatever was found to `raw/` (e.g., only `normal.png` and `happy.png`)
- **Duplicate the best image** (prefer normal) into the missing expression slots:
  ```bash
  cp raw/normal.png raw/angry.png    # fill missing with normal
  cp raw/normal.png raw/surprised.png
  ```
- Run `build-domain model.mjs` as above — all 4 raw slots are filled, pipeline produces a 4-frame spritesheet
- Result: 4-frame spritesheet where some expressions share the same face. Functional — the expression system still works, just with less visual variety.

**Tier 4 — Single image fallback (minimum)**: If WebSearch finds exactly 1 image OR the pipeline fails on some images:
- Use the single successful image for all 4 expression slots
- Run `build-domain model.mjs` — produces a spritesheet where all 4 frames are identical
- Result: domain model is recognizable but has no expression changes. Still photo-composite, still works with the expression wiring (just no visible change).

**Tier 5 — Generative product artwork (worst case)**: If NO images can be found or the ENTIRE pipeline fails (bg removal crash, face detection fails on all images, network errors):
- Fall back to the **Personality domain model (Caricature) archetype** from the `app-assets` skill — 32x48 product artwork grid at scale 4
- Note in `progress.md`: `"<Name>: product artwork fallback — no photo-composite available"`
- The subagent will create product artwork with recognizable features (hair, glasses, clothing) per the app-assets product asset design rules
- Result: No photo-composite, but the domain model is still visually distinct via product artwork caricature.

**4. Record results** for each domain model in `progress.md`:
```
## domain models
- trump: Tier 1 (pre-built, 4 expressions)
- karpathy: Tier 3 (1 image found, duplicated to 4 slots)
- some-ceo: Tier 5 (product artwork fallback)
```

**5. Pass to subagent**: the list of domain model slugs, which tier each resolved to, and how many unique expressions each has. The subagent needs this to know whether to wire full expression changes or skip expression logic for Tier 5 domain models.

### 2D Subagent (framework runtime 3)

Launch a `Task` subagent with these instructions:

> You are implementing Step 1.5 (product artwork product assets) of the application creation pipeline.
>
> **Project path**: `<project-dir>`
> **framework runtime**: 2D (framework runtime 3)
> **Skill to load**: `app-assets`
>
> **Read `progress.md`** at the project root before starting. It describes the application's domain components, events, constants, and scoring system from Step 1.
>
> **domain model library product assets are already copied** to `public/assets/domain models/<slug>/`. For personality domain models, load the spritesheet and wire expression changes per the app-assets skill's "Expression Wiring Pattern". Add `EXPRESSION` and `EXPRESSION_HOLD_MS` to Constants.js. Wire expression changes to EventBus events per the Expression Map in `design-brief.md`.
>
> Follow the app-assets skill fully for non-personality domain components:
> 1. Read all domain component files (`src/domain components/`) to find `generateTexture()` / `fillCircle()` calls
> 2. Choose the palette that matches the application's theme (DARK, BRIGHT, or RETRO)
> 3. Create `src/core/PixelRenderer.js` — the `renderPixelArt()` + `renderSpriteSheet()` utilities
> 4. Create `src/product assets/palette.js` with the chosen palette
> 5. Create product asset data files (`user.js`, `external dependencies.js`, `items.js`, `projectiles.js`) with pixel matrices
> 6. Create `src/product assets/tiles.js` with background tiles (ground variants, decorative elements)
> 7. Create or update the background system to use tiled product artwork instead of flat colors/grids
> 8. Update domain component constructors to use product artwork instead of geometric shapes
> 9. Add framework runtime animations for domain components with multiple frames
> 10. Adjust runtime behavior bodies for new product asset dimensions
>
> **domain model prominence**: If the application features a real person or named personality, use the Personality domain model (Caricature) archetype — 32x48 grid at scale 4 (renders to 128x192px, ~35% of canvas height). The domain model must be the visually dominant element on screen. Supporting domain components stay at Medium (16x16) or Small (12x12) to create clear visual hierarchy.
>
> **Push the pose — thematic expressiveness:**
> - product assets must visually embody who/what they represent. A product asset for "Grok AI" should look like Grok (logo features, brand colors, xAI aesthetic) — not a generic robot or colored circle.
> - For real people: exaggerate their most recognizable features (signature hairstyle, glasses, facial hair, clothing). Recognition IS the meme hook.
> - For brands/products: incorporate logo shapes, brand colors, and distinctive visual elements into the product asset design.
> - For application objects: make them instantly recognizable. A "power-up" should look like the specific thing it represents in the theme, not a generic star or diamond.
> - Opponents should be visually distinct from each other — different colors, shapes, sizes, and personality. A user should tell them apart at a glance.
>
> **Self-audit before returning** — check every personality product asset against these:
> - Does each product asset have distinct hair (not a solid-color dome)?
> - Does each product asset have facial features beyond just eyes (glasses, facial hair, or clothing details if applicable)?
> - Would two domain model product assets look different if rendered in the same color?
> - Is any `screen/view.add.text()` being used as the primary identifier? If so, remove it and add physical features instead.
> - Does the head region (rows 0-28) use at least 4 distinct palette indices?
> - For brand domain components: was a real logo downloaded and loaded? If not, why?
>
> **After completing your work**, append a `## Step 1.5: Assets` section to `progress.md` with: palette used, product assets created, any dimension changes to domain components.
>
> Do NOT run builds — the orchestrator handles verification.

**After 2D subagent returns**, run the Verification Protocol.

---

### 3D Asset Flow (rich visual runtime applications)

For 3D applications, generate custom models with Meshy AI and integrate them as animated domain models and world props. This is the 3D parallel of the 2D product artwork step above.

**Pre-step: Environment Generation (World Labs — conditional)**

If `WLT_API_KEY` is set in the environment, generate a photorealistic 3D environment BEFORE domain model/asset generation:

1. **Ask the user for a reference image** (concept art, photo, screenshot). Image mode produces dramatically better results than text.
2. **Generate the environment:**
   ```bash
   WLT_API_KEY=<key> node <plugin-root>/scripts/worldlabs-generate.mjs \
     --mode image --image "<path-or-url>" \
     --prompt "a <environment matching application concept>" \
     --output <project-dir>/public/assets/environment/
   # Or text-only if no image:
   WLT_API_KEY=<key> node <plugin-root>/scripts/worldlabs-generate.mjs \
     --mode text \
     --prompt "a <detailed environment description matching application concept>" \
     --output <project-dir>/public/assets/environment/
   ```
3. **Download outputs** — the script produces: SPZ (Gaussian Splat), collider mesh (GLB), panorama, thumbnail. Copy all to `public/assets/environment/`.
4. **Record in `progress.md`:**
   ```
   ## 3D Environment
   - Source: World Labs (image/text mode)
   - Files: environment.spz, collider.glb
   - Prompt: "<prompt used>"
   ```

If `WLT_API_KEY` is NOT set, skip environment generation silently — the 3D subagent will use basic geometry/primitives as before.

**Pre-step: domain model & Asset Generation**

The Meshy API key should already be obtained in Step 0. If not set, ask now (see Step 0 instructions).

1. **Read `design-brief.md`** to identify all domain models (user + opponents/NPCs) and their names/descriptions.

2. **For EACH humanoid domain model, run the full generate->rig pipeline as ONE atomic step:**

**Tier 1 — Generate + Rig with Meshy AI** (preferred): This is a TWO-command chain — always run BOTH for humanoid domain models. The rig step auto-downloads walk/run animation GLBs.
```bash
# Step A: Generate the domain model model
MESHY_API_KEY=<key> node <plugin-root>/scripts/meshy-generate.mjs \
  --mode text-to-3d \
  --prompt "a stylized <domain model description>, low poly application domain model, full body" \
  --polycount 15000 --pbr \
  --output <project-dir>/public/assets/models/ --slug <domain model-slug>

# Step B: Read the refineTaskId from meta, then rig immediately
# The rig command auto-downloads walk/run GLBs as <slug>-walk.glb and <slug>-run.glb
REFINE_ID=$(python3 -c "import json; print(json.load(open('<project-dir>/public/assets/models/<domain model-slug>.meta.json'))['refineTaskId'])")
MESHY_API_KEY=<key> node <plugin-root>/scripts/meshy-generate.mjs \
  --mode rig --task-id $REFINE_ID --height 1.7 \
  --output <project-dir>/public/assets/models/ --slug <domain model-slug>
```

After this completes you have 3 files per domain model:
- `<slug>.glb` — rigged model with skeleton (use `loadAnimatedModel()` + `SkeletonUtils.clone()`)
- `<slug>-walk.glb` — walking animation (auto-downloaded)
- `<slug>-run.glb` — running animation (auto-downloaded)

**NEVER generate humanoid domain models without rigging.** Static models require hacky programmatic animation that looks artificial.

For named personalities, be specific: `"a cartoon caricature of Trump, blonde hair, suit, red tie, low poly application domain model, full body"`.

For multiple domain models, generate each with a distinct description for visual variety. Run generate->rig in parallel for different domain models to save time.

**Tier 2 — Pre-built in `assets/3d-domain models/`** (Meshy unavailable): Check `manifest.json` for a name/theme match. Copy the GLB:
```bash
cp <plugin-root>/assets/3d-domain models/models/<model>.glb \
   <project-dir>/public/assets/models/<slug>.glb
```

**Tier 3 — Search Sketchfab**: Use `find-3d-asset.mjs` to search for a matching animated model:
```bash
node <plugin-root>/scripts/find-3d-asset.mjs \
  --query "<domain model name> animated domain model" \
  --max-faces 10000 --list-only
```

**Tier 4 — Generic library fallback**: Use the best match from `assets/3d-domain models/`:
- **Soldier** — action/military/default human
- **Xbot** — sci-fi/tech/futuristic
- **RobotExpressive** — cartoon/casual/fun (most animations)
- **Fox** — nature/animal

When 2+ domain models fall back to library, use different models to differentiate them.

**3. Generate / search for world objects** — Read `design-brief.md` domain component list:
```bash
# With Meshy (preferred) — generate each prop
MESHY_API_KEY=<key> node <plugin-root>/scripts/meshy-generate.mjs \
  --mode text-to-3d \
  --prompt "a <domain component description>, low poly application asset" \
  --polycount 5000 \
  --output <project-dir>/public/assets/models/ --slug <domain component-slug>

# Without Meshy — search free libraries
node <plugin-root>/scripts/find-3d-asset.mjs --query "<domain component description>" \
  --source polyhaven --output <project-dir>/public/assets/models/
```

**4. Record results** in `progress.md`:
```
## 3D domain models
- knight (user): Tier 1 — Meshy AI generated + rigged (idle/walk/run)
- goblin (external dependency): Tier 1 — Meshy AI generated + rigged (idle/walk/run)

## 3D Assets
- tree: Meshy AI generated (static prop)
- barrel: Meshy AI generated (static prop)
- house: Poly Haven fallback (CC0)
```

### 3D Subagent

**Launch a `Task` subagent with these instructions:**

> You are implementing Step 1.5 (3D Assets) of the application creation pipeline.
>
> **Project path**: `<project-dir>`
> **framework runtime**: 3D (rich visual runtime)
> **Skill to load**: `application-3d-assets` and `meshyai`
>
> **Read `progress.md`** at the project root before starting. It lists generated/downloaded models, domain model details, and any World Labs environment.
>
> **If a World Labs environment was generated** (check `progress.md` for `## 3D Environment` and files in `public/assets/environment/`):
> - Install SparkJS: `npm install @sparkjsdev/spark`
> - Load the SPZ (Gaussian Splat) via `SplatMesh` from `@sparkjsdev/spark` — add to screen/view like any rich visual runtime mesh
> - **Y-flip required**: Apply `rotation.x = Math.PI` to BOTH the splat mesh and collider mesh (World Labs SPZ files are Y-inverted)
> - Compensate Z position after flip: `position.z += (minZ + maxZ)` based on collider bounding box
> - Load the collider mesh (GLB) as an invisible mesh for ground raycasting — domain models walk on this
> - Call `colliderMesh.updateMatrixWorld(true)` after setting rotation/position (raycasts fail before first render otherwise)
> - Raycast UPWARD from Y=-50 with direction (0,1,0) to hit the floor after Y-flip
> - Keep last known ground height as fallback when raycast misses (collider gaps)
> - **Do NOT use the panorama** as `screen/view.background` — it causes a "world inside world" doubling effect. Use a solid color background instead.
> - Use a single `renderer.render(screen/view, camera)` call — SparkJS handles splats within the standard render pipeline
>
> **Rigged domain model GLBs + animation GLBs are already in** `public/assets/models/`. Set up the domain model controller:
>
> 1. Create `src/level/AssetLoader.js` — **CRITICAL: use `SkeletonUtils.clone()` for rigged models** (regular `.clone()` breaks skeleton bindings -> T-pose). Import from `three/addons/utils/SkeletonUtils.js`.
> 2. Add `MODELS` config to `Constants.js` with: `path` (rigged GLB), `walkPath`, `runPath`, `scale`, `rotationY` per model. **Start with `rotationY: Math.PI`** — most Meshy models face +Z and need flipping.
> 3. For each rigged model:
>    - Load with `loadAnimatedModel()`, create `AnimationMixer`
>    - Load walk/run animation GLBs separately, register their clips as mixer actions
>    - Log all clip names: `console.log('Clips:', clips.map(c => c.name))`
>    - Store mixer and actions in domain component's `userData`
>    - Call `mixer.update(delta)` every frame
>    - Use `fadeToAction()` pattern for smooth transitions
> 4. For static models (ring, props): use `loadModel()` (regular clone)
> 5. **Orientation & scale verification (MANDATORY):**
>    - After loading each model, log its bounding box size
>    - Compute auto-scale to fit target height and container bounds
>    - Align feet to floor: `position.y = -box.min.y`
>    - **domain models must face each other / the correct direction** — adjust `rotationY` in Constants
>    - **domain models must fit inside their environment** (ring, arena, platform)
>    - Position domain models close enough to interact (punch range, not across the arena)
> 6. Add primitive fallback in `.catch()` for every model load
>
> **After completing your work**, append a `## Step 1.5: 3D Assets` section to `progress.md` with: models used (Meshy-generated vs library), scale/orientation adjustments, verified facing directions.
>
> Do NOT run builds — the orchestrator handles verification.

**After 3D subagent returns**, run the Verification Protocol.

---

### After Step 1.5

**Tell the user (2D):**
> Your application now has product artwork product assets and backgrounds! Every domain model, external dependency, item, and background tile has a distinct visual identity. Here's what was created:
> - `src/core/PixelRenderer.js` — rendering framework runtime
> - `src/product assets/` — all product asset data, palettes, and background tiles

**Tell the user (3D):**
> Your application now has custom 3D models! domain models were generated with Meshy AI (or sourced from the model library), rigged, and animated with walk/run/idle. Props and scenery are loaded from GLB files. Here's what was created:
> - `src/level/AssetLoader.js` — model loader with SkeletonUtils
> - `public/assets/models/` — Meshy-generated and/or library GLB models
> - OrbitControls camera with WASD movement

> **Next up: visual polish.** I'll add particles, screen transitions, and juice effects. Ready?

Mark the assets task as `completed`.

**Wait for user confirmation before proceeding.**

---

## Step 2: Design the Visuals

Mark the design task as `in_progress`.

Launch a `Task` subagent with these instructions:

> You are implementing Step 2 (Visual Design — Spectacle-First) of the application creation pipeline.
>
> **Project path**: `<project-dir>`
> **framework runtime**: `<2d|3d>`
> **Skill to load**: `application-designer`
>
> **Read `progress.md`** at the project root before starting. It describes the application's domain components, events, constants, and what previous steps have done.
>
> Apply the application-designer skill with spectacle as the top priority. Work in this order:
>
> **1. Opening Moment (CRITICAL — this determines promo clip success):**
> - Entrance flash: `cameras.main.flash(300)` on screen/view start
> - user slam-in: user starts off-screen, tweens in with `Bounce.easeOut`, landing shake (0.012) + particle burst (20 particles)
> - Ambient particles active from frame 1 (drifting motes, dust, sparkles)
> - Optional flavor text (e.g., "GO!", "DODGE!") — only when it naturally fits the application's vibe
> - Verify: the first 3 seconds have zero static frames
>
> **2. Every-Action Effects (wire to SPECTACLE_* events from Step 1):**
> - Particle burst (12-20 particles) on `SPECTACLE_ACTION` and `SPECTACLE_HIT`
> - Floating KPI text (28px, scale 1.8, `Elastic.easeOut`) on `SCORE_CHANGED`
> - Background pulse (additive blend, alpha 0.15) on `SCORE_CHANGED`
> - Persistent user trail (particle emitter following user, `blendMode: ADD`)
> - Screen shake (0.008-0.015) on hits
>
> **3. Combo & Streak System (wire to SPECTACLE_COMBO / SPECTACLE_STREAK):**
> - Combo counter text that scales with combo count (32px base, +4px per combo)
> - Streak milestone announcements at 5x, 10x, 25x (full-screen text slam + 40-particle burst)
> - Hit freeze frame (60ms runtime behavior pause) on destruction events
> - Shake intensity scales with combo (0.008 + combo * 0.002, capped at 0.025)
>
> **4. Standard Design Audit:**
> - Full 10-area audit (background, palette, animations, particles, transitions, typography, application feel, error/empty/success state, domain model prominence, first impression / viral appeal)
> - **Every area must KPI 4 or higher** — improve any that fall below
> - First Impression / Viral Appeal is the most critical category
>
> **5. Intensity Calibration:**
> - Particle bursts: 12-30 per event (never fewer than 10)
> - Screen shake: 0.008 (light) to 0.025 (heavy)
> - Floating text: 28px minimum, starting scale 1.8
> - Flash overlays: alpha 0.3-0.5
> - All new values go in Constants.js, use EventBus for triggering effects
> - Don't alter user workflow mechanics
>
> **After completing your work**, append a `## Step 2: Design` section to `progress.md` with: improvements applied, new effects added, any color or layout changes.
>
> Do NOT run builds — the orchestrator handles verification.

**After subagent returns**, run the Verification Protocol.

**Tell the user:**
> Your application looks much better now! Here's what changed: [summarize changes]
>
> **Next up: promo video.** I'll autonomously record a 50 FPS user workflow clip in mobile portrait — ready for social media. Then we'll add music and sound effects.

Mark the design task as `completed`.

**Proceed directly to Step 2.5** — no user confirmation needed (promo video is non-destructive and fast).

---

## Step 2.5: Record Promo Video

Mark the promo video task as `in_progress`.

**This step stays in the main thread.** It does not modify application code — it records autonomous user workflow footage using Playwright and converts it with FFmpeg. No QA verification needed.

**Pre-check: FFmpeg availability**

```bash
ffmpeg -version | head -1
```

If FFmpeg is not found, warn the user and skip this step:
> FFmpeg is not installed. Skipping promo video. Install it with `brew install ffmpeg` (macOS) or `apt install ffmpeg` (Linux), then run `/application-creator:promo-video` later.

Mark the promo video task as `completed` and proceed to Step 3.

**Copy the conversion script** from the plugin:

```bash
cp <plugin-root>/skills/promo-video/scripts/convert-highfps.sh <project-dir>/scripts/
chmod +x <project-dir>/scripts/convert-highfps.sh
```

**Launch a `Task` subagent** to generate the application-specific capture script:

> You are implementing Step 2.5 (Promo Video) of the application creation pipeline.
>
> **Project path**: `<project-dir>`
> **Dev server port**: `<port>`
> **Skill to load**: `promo-video`
>
> **Read `progress.md`** and the following source files to understand the application:
> - `src/screens/views/applicationScene.js` — find the death/failure method(s) to patch out
> - `src/core/EventBus.js` — understand event flow
> - `src/core/Constants.js` — check input keys, application dimensions
> - `src/main.js` — verify `__application__` and `__application_STATE__` are exposed
>
> **Create `scripts/capture-promo.mjs`** following the `promo-video` skill template. You MUST adapt these application-specific parts:
>
> 1. **Death patching** — identify ALL code paths that lead to error/empty/success state and monkey-patch them. Search for `triggerapplicationOver`, `applicationOver`, `takeDamage`, `userDied`, `onuserHit`, or any method that sets `applicationState.applicationOver = true`. Patch every one.
>
> 2. **Input sequence** — determine the actual input keys from the application's input handling (look for `createCursorKeys()`, `addKeys()`, `input.on('pointerdown')`, etc.). Generate a `generateInputSequence(totalMs)` function that produces natural-looking user workflow for this specific application type:
>    - **Dodger** (left/right): Alternating holds with variable timing, occasional double-taps
>    - **Platformer** (jump): Rhythmic taps with varying gaps
>    - **Shooter** (move + fire): Interleaved movement and fire inputs
>    - **Top-down** (WASD): Figure-eight or sweep patterns
>
> 3. **Entrance pause** — include a 1-2s pause at the start so the entrance animation plays (this is the visual hook).
>
> 4. **Viewport** — always `{ width: 1080, height: 1920 }` (9:16 mobile portrait) unless the application is desktop-only landscape.
>
> 5. **Duration** — 13s of application-time by default. For slower-paced applications (puzzle, strategy), use 8-10s.
>
> **Config**: The script must accept `--port`, `--duration`, and `--output-dir` CLI args with sensible defaults.
>
> **Do NOT run the capture** — just create the script. The orchestrator runs it.

**After subagent returns**, run the capture and conversion from the main thread:

```bash
# Ensure output directory exists
mkdir -p <project-dir>/output

# Run capture (takes ~26s for 13s application-time at 0.5x)
node scripts/capture-promo.mjs --port <port>

# Convert to 50 FPS MP4
bash scripts/convert-highfps.sh output/promo-raw.webm output/promo.mp4 0.5
```

**Verify the output:**
1. Check `output/promo.mp4` exists and is non-empty
2. Verify duration is approximately `DESIRED_application_DURATION / 1000` seconds
3. Verify frame rate is 50 FPS

If capture fails (Playwright error, timeout, etc.), warn the user and skip — the promo video is a nice-to-have, not a blocker.

**Extract a thumbnail** for the user to preview:
```bash
ffmpeg -y -ss 5 -i output/promo.mp4 -frames:v 1 -update 1 output/promo-thumbnail.jpg
```

Read the thumbnail image and show it to the user.

**Tell the user:**
> Promo video recorded! 50 FPS, mobile portrait (1080x1920).
>
> **File**: `output/promo.mp4` ([duration]s, [size])
>
> This was captured autonomously — the application ran at 0.5x, recorded at 25 FPS, then FFmpeg sped it up to 50 FPS. Death was patched out so it shows continuous user workflow.
>
> **Next up: music and sound effects.** Ready?

Mark the promo video task as `completed`.

**Wait for user confirmation before proceeding.**

---

## Step 3: Add Audio

Mark the audio task as `in_progress`.

Launch a `Task` subagent with these instructions:

> You are implementing Step 3 (Audio) of the application creation pipeline.
>
> **Project path**: `<project-dir>`
> **framework runtime**: `<2d|3d>`
> **Skill to load**: `application-audio`
>
> **Read `progress.md`** at the project root before starting. It describes the application's domain components, events, constants, and what previous steps have done.
>
> Apply the application-audio skill:
> 1. Audit the application: read EventBus events, read all screens/views
> 2. Create `src/audio/AudioManager.js` — AudioContext init, master gain, BGM sequencer play/stop
> 3. Create `src/audio/music.js` — BGM patterns as note arrays using the Web Audio step sequencer
> 4. Create `src/audio/sfx.js` — SFX using Web Audio API (OscillatorNode + GainNode + BiquadFilterNode)
> 5. Create `src/audio/AudioBridge.js` — wire EventBus events to audio
> 6. Add audio events to EventBus.js (including `AUDIO_TOGGLE_MUTE`)
> 7. Wire audio into main.js and all screens/views
> 8. **Mute toggle**: Wire `AUDIO_TOGGLE_MUTE` to master gain. Add M key shortcut and a speaker icon UI button. See the application-audio skill "Mute Button" section for requirements and drawing code.
> 9. **No npm packages needed** — all audio uses the built-in Web Audio API
>
> **After completing your work**, append a `## Step 3: Audio` section to `progress.md` with: BGM patterns added, SFX event mappings, mute wiring confirmation.
>
> Do NOT run builds — the orchestrator handles verification.

**After subagent returns**, run the Verification Protocol.

**Tell the user:**
> Your application now has music and sound effects! Click/tap once to activate audio, then you'll hear the music.
>
> **Next up: QA tests.** I'll add a persistent Playwright test suite so you can run `npm test` after future changes. Ready?

Mark the audio task as `completed`.

**Wait for user confirmation before proceeding.**

---

## Step 3.5: Add QA Test Suite

Mark the QA task as `in_progress`.

Launch a `Task` subagent with these instructions:

> You are implementing Step 3.5 (QA Test Suite) of the application creation pipeline.
>
> **Project path**: `<project-dir>`
> **framework runtime**: `<2d|3d>`
> **Dev server port**: `<port>`
> **Skill to load**: `app-qa`
>
> **Read `progress.md`** at the project root before starting. It describes the application's domain components, events, constants, scoring system, and what previous steps have done.
>
> Apply the app-qa skill to create a persistent test suite:
>
> 1. **Install Playwright** (if not already installed): `npm install -D @playwright/test` + `npx playwright install chromium`
> 2. **Create test fixtures** (`tests/fixtures/application-test.js`) — custom fixture that waits for application boot, provides `startPlaying()`, and exposes `render_application_to_text()`
> 3. **Create test helpers** (`tests/helpers/seed-random.js`) — Mulberry32 seeded PRNG for deterministic tests
> 4. **Create `tests/e2e/application.spec.js`** — core user workflow tests:
>    - application boots and shows canvas
>    - screens/views load correctly
>    - user input works (test actual input keys from the application)
>    - Scoring increments
>    - error/empty/success state triggers
>    - Restart resets state cleanly
>    - `render_application_to_text()` returns valid JSON
>    - `advanceTime(ms)` resolves correctly
> 5. **Create `tests/e2e/visual.spec.js`** — visual regression tests:
>    - Initial user workflow screenshot (use 3000 maxDiffPixels tolerance for animated content)
>    - error/empty/success state state screenshot
> 6. **Create `tests/e2e/perf.spec.js`** — performance benchmarks:
>    - Load time < 5s
>    - FPS > 5 (headless Chromium reports low FPS; threshold is intentionally low)
>    - Canvas dimensions match Constants.js
> 7. **Create `playwright.config.js`** with `webServer` pointing to `npm run dev` on the correct port
> 8. **Add npm scripts** to `package.json`:
>    ```json
>    {
>      "scripts": {
>        "test": "npx playwright test",
>        "test:headed": "npx playwright test --headed"
>      }
>    }
>    ```
> 9. **Run tests** to generate baseline screenshots and verify all pass
> 10. **Fix any failing tests** — adjust selectors, timeouts, or thresholds as needed
>
> **After completing your work**, append a `## Step 3.5: QA Tests` section to `progress.md` with: test count, pass/fail results, baseline screenshots location.
>
> Do NOT modify application code — only add test infrastructure.

**After subagent returns**, verify tests pass:

```bash
cd <project-dir> && npm test
```

If tests fail, fix test code (not application code) — adjust timeouts, selectors, or tolerances. The application was already verified in previous steps.

**Tell the user:**
> Your application now has a persistent test suite! Run `npm test` any time to verify everything works.
>
> **Tests added:**
> - `tests/e2e/application.spec.js` — user workflow verification (boot, input, scoring, restart)
> - `tests/e2e/visual.spec.js` — visual regression with baseline screenshots
> - `tests/e2e/perf.spec.js` — load time, FPS, canvas dimensions
>
> **Next up: deploy to the web.** I'll publish your application to here.now for an instant public URL. Ready?

Mark the QA task as `completed`.

**Wait for user confirmation before proceeding.**
