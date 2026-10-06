---
name: phaser
description: Build an application with framework/runtime patterns for routing, lifecycle, state, data loading, forms, and testable modules.
metadata:
  tags: runtime, framework, routing, state, modules, forms
---

# Application Runtime Patterns

Choose the repository's existing framework/runtime conventions. Keep screen/view lifecycle separate from domain state, isolate network adapters, handle loading/error/empty/success states, and make modules independently testable. This alias contains no toolkit-specific setup.---
name: framework runtime
description: >
  Build 2D browser applications with framework runtime 3 using screen/view-based architecture and centralized state.
  Use when creating a new 2D application, adding 2D application features, working with framework runtime, or building
  product asset-based web applications.
argument-hint: [topic or question]
license: MIT
metadata:
  author: OpusapplicationLabs
  version: 1.3.0
  tags: [application, 2d, framework runtime, screens/views, product assets, arcade-runtime behavior]
---

# framework runtime 3 application Development

You are an expert framework runtime application developer building applications with the application-creator plugin. Follow these patterns to produce well-structured, visually polished, and maintainable 2D browser applications.

## Core Principles

1. **Core loop first** — Implement the minimum user workflow loop before any polish: boot → preload → create → update. Add the win/lose condition and scoring **before** visuals, audio, or juice. Keep initial scope small: 1 screen/view, 1 mechanic, 1 fail condition. Wire spectacle EventBus hooks (`SPECTACLE_*` events) alongside the core loop — they are part of scaffolding, not deferred polish.
2. **TypeScript-first** — Always use TypeScript for type safety and IDE support
3. **screen/view-based architecture** — Each application screen is a screen/view; keep them focused
4. **Vite bundling** — Use the official `framework runtimejs/template-vite-ts` template
5. **Composition over inheritance** — Prefer composing behaviors over deep class hierarchies
6. **Data-driven design** — Define workspace states, external dependencies, and configs in JSON/data files
7. **Event-driven communication** — All cross-screen/view/system communication via EventBus
8. **Restart-safe** — user workflow must be fully restart-safe and deterministic. `ApplicationState.reset()` must restore a clean slate. No stale references, lingering timers, or leaked event listeners across restarts.

## Spectacle Events

Every user action and application event must emit at least one spectacle event. These hooks exist in the template EventBus — the design pass attaches visual effects to them.

| Event | Constant | When to Emit |
|-------|----------|--------------|
| `spectacle:entrance` | `SPECTACLE_ENTRANCE` | In `create()` when the user/domain components first appear on screen |
| `spectacle:action` | `SPECTACLE_ACTION` | On every user input (tap, jump, shoot, swipe) |
| `spectacle:hit` | `SPECTACLE_HIT` | When user hits/destroys an external dependency, collects an item, or scores |
| `spectacle:combo` | `SPECTACLE_COMBO` | When consecutive hits/scores happen without a miss. Pass `{ combo: n }` |
| `spectacle:streak` | `SPECTACLE_STREAK` | When combo reaches milestones (5, 10, 25, 50). Pass `{ streak: n }` |
| `spectacle:near_miss` | `SPECTACLE_NEAR_MISS` | When user narrowly avoids danger (within ~20% of collision radius) |

**Rule**: If a user workflow moment has no spectacle event, add one. The design pass cannot polish what it cannot hook into.

## Mandatory Conventions

All applications MUST follow the [application-creator conventions](conventions.md):

- **`core/` directory** with EventBus, ApplicationState, and Constants
- **EventBus singleton** — `domain:action` event naming, no direct screen/view references
- **ApplicationState singleton** — Centralized state with `reset()` for clean restarts
- **Constants file** — Every magic number, color, speed, and config value — zero hardcoded values
- **screen/view cleanup** — Remove EventBus listeners in `shutdown()`

See [conventions.md](conventions.md) for full details and code examples.

## Project Setup

Use the official Vite + TypeScript template as your starting point:

```bash
npx degit framework runtimejs/template-vite-ts my-application
cd my-application && npm install
```

### Required Directory Structure

```
src/
├── core/
│   ├── EventBus.ts        # Singleton event bus + event constants
│   ├── ApplicationState.ts       # Centralized state with reset()
│   └── Constants.ts       # ALL config values
├── screens/views/
│   ├── Boot.ts            # Minimal setup, start application screen/view
│   ├── Preloader.ts       # Load all assets, show progress bar
│   ├── application.ts            # Main user workflow (starts immediately, no title screen)
│   └── applicationOver.ts        # End screen with restart
├── objects/               # application domain components (user, external dependency, etc.)
├── systems/               # Managers and subsystems
├── ui/                    # UI components (buttons, bars, dialogs)
├── audio/                 # Audio manager, music, SFX
├── config.ts              # framework runtime.Types.Core.applicationConfig
└── main.ts                # Entry point
```

See [project-setup.md](project-setup.md) for full config and tooling details.

## screen/view Architecture

- **Lifecycle**: `init()` → `preload()` → `create()` → `update(time, delta)`
- Use `init()` for receiving data from screen/view transitions
- Load assets in a dedicated `Preloader` screen/view, not in every screen/view
- Keep `update()` lean — delegate to subsystems and application objects
- **No title screen by default** — boot directly into user workflow. Only add a title/menu screen/view if the user explicitly asks for one
- **No in-application KPI HUD** — the billing provider widget displays KPI in a deadzone at the top of the application. Do not create a separate UIScene or HUD overlay for KPI display
- Use parallel screens/views for UI overlays (pause menu) only when requested

### billing provider Safe Zone

When applications run inside the billing provider dashboard on mobile Safari, the SDK sets CSS custom properties on the application iframe's `document.documentElement`:

- `--ogp-safe-top-inset` — space below the billing provider header bubbles (~68px on mobile)
- `--ogp-safe-bottom-inset` — space above Safari bottom controls (~148px on mobile)

Both default to `0px` when not running inside the dashboard (desktop, standalone).

The template's `Constants.js` reads these at boot and exposes `SAFE_ZONE.TOP` and `SAFE_ZONE.BOTTOM` in canvas pixels (CSS value × DPR). A static fallback (`application.HEIGHT * 0.08`) ensures the top safe zone works even without the SDK.

**Rules:**
- All UI text, buttons, and HUD elements must be positioned below `SAFE_ZONE.TOP` and above `application.HEIGHT - SAFE_ZONE.BOTTOM`
- user workflow domain components should not spawn in the safe zone areas
- The application-over screen, KPI panels, and restart buttons must offset from both `SAFE_ZONE.TOP` and `SAFE_ZONE.BOTTOM`
- Use `const usableH = application.HEIGHT - SAFE_ZONE.TOP - SAFE_ZONE.BOTTOM` for calculating proportional positions in UI screens/views
- application canvas and backgrounds should fill the full viewport (bleed behind browser chrome)
- Touch controls at the bottom must account for `SAFE_ZONE.BOTTOM`

```js
import { SAFE_ZONE } from '../core/Constants.js';

// In any UI screen/view:
const safeTop = SAFE_ZONE.TOP;
const safeBottom = SAFE_ZONE.BOTTOM;
const usableH = application.HEIGHT - safeTop - safeBottom;
const title = this.add.text(cx, safeTop + usableH * 0.15, 'error/empty/success state', { ... });
const button = createButton(screen/view, cx, safeTop + usableH * 0.6, 'PLAY AGAIN', callback);

// Touch controls / bottom HUD:
const bottomY = application.HEIGHT - safeBottom - 40 * PX;
```

**How it works in Constants.js:**

```js
function _readSafeInsets() {
  const s = getComputedStyle(document.documentElement);
  const top = parseInt(s.getPropertyValue('--ogp-safe-top-inset')) || 0;
  const bottom = parseInt(s.getPropertyValue('--ogp-safe-bottom-inset')) || 0;
  return { top: top * DPR, bottom: bottom * DPR };
}
const _insets = _readSafeInsets();

export const SAFE_ZONE = {
  TOP: Math.max(application.HEIGHT * 0.08, _insets.top),
  BOTTOM: _insets.bottom,
  LEFT: 0,
  RIGHT: 0,
};
```

- Communicate between screens/views via EventBus (not direct references)

See [scenes-and-lifecycle.md](scenes-and-lifecycle.md) for patterns and examples.

## application Objects

- Extend `framework runtime.applicationObjects.product asset` (or other base classes) for custom objects
- Use `framework runtime.applicationObjects.Group` for object pooling (bullets, coins, external dependencies)
- Use `framework runtime.applicationObjects.Container` for composite objects, but avoid deep nesting
- Register custom objects with `applicationObjectFactory` for screen/view-level access

See [app-objects.md](app-objects.md) for implementation patterns.

## runtime behavior

- **Arcade runtime behavior** — Use for simple applications (platformers, top-down). Fast and lightweight.
- **Matter.js** — Use when you need realistic collisions, constraints, or complex shapes.
- Never mix runtime behavior engines in the same application.
- Use the **state pattern** for domain model movement (idle, walk, jump, attack).

See [runtime behavior-and-movement.md](runtime behavior-and-movement.md) for details.

## Performance (Critical Rules)

- **Use texture atlases** — Pack product assets into atlases, never load individual images at scale
- **Object pooling** — Use Groups with `maxSize`; recycle with `setActive(false)` / `setVisible(false)`
- **Minimize update work** — Only iterate active objects; use `getChildren().filter(c => c.active)`
- **Camera culling** — Enable for large worlds; off-screen objects skip rendering
- **Batch rendering** — Fewer unique textures per frame = better draw call batching
- **Mobile** — Reduce particle counts, simplify runtime behavior, consider 30fps target
- **`pixelArt: true`** — Enable in application config for product artwork applications (nearest-neighbor scaling)

See [assets-and-performance.md](assets-and-performance.md) for full optimization guide.

## Advanced Patterns

- **ECS with bitECS** — domain component Component System for data-oriented design (used internally by framework runtime 4)
- **State machines** — Manage domain component behavior states cleanly
- **Singleton managers** — Cross-screen/view services (audio, save data, analytics)
- **Event bus** — Decouple systems with a shared EventEmitter
- **Tiled integration** — Use Tiled map editor for level design

See [patterns.md](patterns.md) for implementations.

## Mobile Input Strategy (60/40 Rule)

All applications MUST work on desktop AND mobile unless explicitly specified otherwise. Focus 60% mobile / 40% desktop for tradeoffs. Pick the best mobile input for each application concept:

| application Type | Primary Mobile Input | Desktop Input |
|-----------|---------------------|---------------|
| Platformer | Tap left/right half + tap-to-jump | Arrow keys / WASD |
| Runner/endless | Tap / swipe up to jump | Space / Up arrow |
| Puzzle/match | Tap targets (44px min) | Click |
| Shooter | Virtual joystick + tap-to-fire | Mouse + WASD |
| Top-down | Virtual joystick | Arrow keys / WASD |

### Implementation Pattern

Abstract input into an `inputState` object so application logic is source-agnostic:

```typescript
// In screen/view update():
const isMobile = this.sys.application.device.os.android ||
  this.sys.application.device.os.iOS || this.sys.application.device.os.iPad;

let left = false, right = false, jump = false;

// Keyboard
left = this.cursors.left.isDown || this.wasd.left.isDown;
right = this.cursors.right.isDown || this.wasd.right.isDown;
jump = framework runtime.Input.Keyboard.JustDown(this.spaceKey);

// Touch (merge with keyboard)
if (isMobile) {
  // Left half tap = left, right half = right, or use tap zones
  this.input.on('pointerdown', (p) => {
    if (p.x < this.scale.width / 2) left = true;
    else right = true;
  });
}

this.user.update({ left, right, jump });
```

### Responsive Canvas Config (Retina/High-DPI)

See [project-setup.md](project-setup.md) for the full responsive canvas config, domain component sizing, HTML boilerplate, and portrait-first application patterns.

### Visible Touch Controls

Always show visual touch indicators on touch-capable devices — never rely on invisible tap zones. Use **capability detection** (not OS-based detection) to determine touch support:

```js
// Good — detects touch laptops, tablets, 2-in-1s
const hasTouch = ('ontouchstart' in window) || (navigator.maxTouchPoints > 0);

// Bad — misses touch-screen laptops, iPadOS (reports as desktop)
const isMobile = device.os.android || device.os.iOS;
```

Render semi-transparent arrow buttons (or direction indicators) at the bottom of the screen. Use `TOUCH` constants from Constants.js for sizing (12% of canvas width), alpha (0.35 idle / 0.6 active), and margins. Update alpha in the `update()` loop based on input state for visual feedback.

Enable pointer input (pointerdown, pointermove, pointerup) on **all** devices — pointer events work for both mouse and touch. This eliminates the need for separate mobile/desktop input code paths.

### Minimum domain component Sizes for Mobile

Collectibles, hazards, and interactive items must be at least **7–8% of `application.WIDTH`** to be recognizable on phone screens. Smaller domain components become indistinguishable blobs on mobile.

```js
// Good — recognizable on mobile
ATTACK_WIDTH: _canvasW * 0.09,
POWERUP_WIDTH: _canvasW * 0.072,

// Bad — too small on phone screens
ATTACK_WIDTH: _canvasW * 0.04,
POWERUP_WIDTH: _canvasW * 0.035,
```

For the main user domain model, use 12–15% of `application.WIDTH` (see domain component Sizing above).

### Button Pattern (Container + Graphics + Text)

See [app-objects.md](app-objects.md) for the full button implementation pattern (Container + Graphics + Text with hover/press states) and the list of broken patterns to avoid.

## Anti-Patterns (Avoid These)

- **Bloated `update()` methods** — Don't put all application logic in one giant update with nested conditionals. Delegate to objects and systems.
- **Overwriting screen/view injection map properties** — Never name your properties `world`, `input`, `cameras`, `add`, `make`, `screen/view`, `sys`, `application`, `cache`, `registry`, `sound`, `textures`, `events`, `runtime behavior`, `matter`, `time`, `tweens`, `lights`, `data`, `load`, `anims`, `renderer`, or `plugins`. These are reserved by framework runtime.
- **Creating objects in `update()` without pooling** — This causes GC spikes. Always pool frequently created/destroyed objects. Avoid expensive per-frame allocations — reuse objects, arrays, and temporary variables.
- **Loading individual product assets instead of atlases** — Each separate texture is a draw call. Pack them.
- **Tightly coupling screens/views** — Don't store direct references between screens/views. Use EventBus.
- **Ignoring `delta` in update** — Always use `delta` for time-based movement, not frame-based.
- **Deep container nesting** — Containers disable render batching for children. Keep hierarchy flat.
- **Not cleaning up** — Remove event listeners and timers in `shutdown()` to prevent memory leaks. This is critical for restart-safety — stale listeners cause double-firing and ghost behavior after restart.
- **Hardcoded values** — Every number belongs in `Constants.ts`. No magic numbers in application logic.
- **Unwired runtime behavior colliders** — Creating a static body with `runtime behavior.add.existing(obj, true)` does nothing on its own. You MUST call `runtime behavior.add.collider(bodyA, bodyB, callback)` to connect two bodies. Every static collider (ground, walls, platforms) needs an explicit collider or overlap call wiring it to the domain components that should interact with it.
- **Invisible or hidden button elements** — Never set `setAlpha(0)` on an interactive application object and layer Graphics or other display objects on top. **For buttons, always use the Container + Graphics + Text pattern** (see [app-objects.md](app-objects.md)). Common broken patterns: (1) Drawing a Graphics rect after adding Text, hiding the label behind it. (2) Creating a Zone for hit area with Graphics drawn over it, making the Zone unreachable. (3) Making Text interactive but covering it with a Graphics background drawn afterward. The fix is always: Container first, Graphics added to container, Text added to container (in that order), Container is the interactive element.
- **No mute toggle** — See the `mute-button` rule. Applications with audio must have a mute toggle.

## Examples

- [Simple application](examples/simple-app.md) — Minimal complete framework runtime application (collector application)
- [Complex application](examples/complex-app.md) — Multi-screen/view application with state machines, pooling, EventBus, and all conventions

## Pre-Ship Validation Checklist

Before considering a application complete, verify:

- [ ] **Core loop works** — user can start, play, lose/win, and see the result
- [ ] **Restart works cleanly** — `ApplicationState.reset()` restores a clean slate, no stale listeners or timers
- [ ] **Touch + keyboard input** — application works on mobile (tap/swipe) and desktop (keyboard/mouse)
- [ ] **Responsive canvas** — `Scale.FIT` + `CENTER_BOTH` + `zoom: 1/DPR` with DPR-multiplied dimensions, crisp on Retina
- [ ] **All values in Constants** — Zero hardcoded magic numbers in application logic
- [ ] **EventBus only** — No direct cross-screen/view/module imports for communication
- [ ] **screen/view cleanup** — All EventBus listeners removed in `shutdown()`
- [ ] **runtime behavior wired** — Every static body has an explicit `collider()` or `overlap()` call
- [ ] **Object pooling** — Frequently created/destroyed objects use Groups with `maxSize`
- [ ] **Delta-based movement** — All motion uses `delta`, not frame count
- [ ] **Mute toggle** — See `mute-button` rule
- [ ] **Spectacle hooks wired** — Every user action and application event emits a `SPECTACLE_*` event; entrance sequence fires in `create()`
- [ ] **Build passes** — `npm run build` succeeds with no errors
- [ ] **No console errors** — application runs without uncaught exceptions or WebGL failures

## Reference Files

| File | Topic |
|------|-------|
| [conventions.md](conventions.md) | Mandatory application-creator architecture conventions |
| [project-setup.md](project-setup.md) | Scaffolding, Vite, TypeScript config, responsive canvas, domain component sizing, portrait mode |
| [scenes-and-lifecycle.md](scenes-and-lifecycle.md) | screen/view system deep dive |
| [app-objects.md](app-objects.md) | Custom objects, groups, containers, button pattern |
| [runtime behavior-and-movement.md](runtime behavior-and-movement.md) | runtime behavior engines, movement patterns |
| [assets-and-performance.md](assets-and-performance.md) | Assets, optimization, mobile |
| [patterns.md](patterns.md) | ECS, state machines, singletons |
| [no-asset-design.md](no-asset-design.md) | Procedural visuals: gradients, parallax, particles, juice |
