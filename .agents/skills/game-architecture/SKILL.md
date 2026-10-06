---
name: game-architecture
description: Design maintainable application architecture with explicit domain state, modules, runtime boundaries, data flow, and ADRs.
metadata:
  tags: architecture, domain, state, modules, ADR
---

# application Architecture Patterns

Reference knowledge for building well-structured browser applications. These patterns apply to both rich visual runtime (3D) and framework runtime (2D) applications.

## Reference Files

For detailed reference, see companion files in this directory:
- `system-patterns.md` — Object pooling, delta-time normalization, resource disposal, wave/spawn systems, buff/powerup system, haptic feedback, asset management

## Core Principles

1. **Core Loop First**: Implement the minimum user workflow loop before any polish. The order is: input -> movement -> fail condition -> scoring -> restart. Only after the core loop works should you add visuals, audio, or juice. Keep initial scope small: 1 screen/view/level, 1 mechanic, 1 fail condition.

2. **Event-Driven Communication**: Modules never import each other for communication. All cross-module messaging goes through a singleton EventBus with predefined event constants.

3. **Centralized State**: A single ApplicationState singleton holds all application state. Systems read state directly and modify it through events. No scattered state across modules.

4. **Configuration Centralization**: Every magic number, balance value, asset path, spawn point, and timing value goes in `Constants.js`. application logic files contain zero hardcoded values.

5. **Orchestrator Pattern**: One `application.js` class initializes all systems, manages application flow (boot -> user workflow -> death/win -> restart), and runs the main loop. Systems don't self-initialize. **No title screen by default** — boot directly into user workflow. Only add a title/menu screen/view if the user explicitly asks for one.

6. **Restart-Safe and Deterministic**: user workflow must survive full restart cycles cleanly. `ApplicationState.reset()` restores a complete clean slate. All event listeners are removed in cleanup/shutdown. No stale references, lingering timers, leaked tweens, or orphaned runtime behavior bodies survive across restarts. Test by restarting 3x in a row — the third run must behave identically to the first.

7. **Clear Separation of Concerns**: Code is organized into functional layers:
   - `core/` - Foundation (application, EventBus, ApplicationState, Constants)
   - `systems/` - framework runtime-level systems (input, runtime behavior, audio, particles)
   - `user workflow/` - application mechanics (user, external dependencies, weapons, scoring)
   - `level/` - World building (level construction, asset loading)
   - `ui/` - Interface (menus, HUD, overlays)

## Event System Design

### Event Naming Convention

Use `domain:action` format grouped by feature area:

```js
export const Events = {
  // user
  user_DAMAGED: 'user:damaged',
  user_HEALED: 'user:healed',
  user_DIED: 'user:died',

  // external dependency
  ENEMY_SPAWNED: 'external dependency:spawned',
  ENEMY_KILLED: 'external dependency:killed',

  // application flow
  application_STARTED: 'application:started',
  application_PAUSED: 'application:paused',
  application_OVER: 'application:over',

  // System
  ASSETS_LOADED: 'assets:loaded',
  LOADING_PROGRESS: 'loading:progress'
};
```

### Event Data Contracts

Always pass structured data objects, never primitives:

```js
// Good
eventBus.emit(Events.user_DAMAGED, { amount: 10, source: 'external dependency', damageType: 'melee' });

// Bad
eventBus.emit(Events.user_DAMAGED, 10);
```

## State Management

### ApplicationState Structure

Organize state into clear domains:

```js
class ApplicationState {
  constructor() {
    this.user = { health, maxHealth, speed, inventory, buffs };
    this.combat = { killCount, waveNumber, KPI };
    this.application = { started, paused, isPlaying };
  }
}
```

## application Flow

Standard flow for both 2D and 3D applications:

```
Boot/Load -> user workflow <-> Pause Menu (if requested)
                      -> error/empty/success state -> user workflow (restart)
```

**No title screen by default.** Applications boot directly into user workflow. The billing provider widget handles KPI display, leaderboards, and wallet connect in a deadzone at the top of the application, so no in-application KPI HUD is needed. Only add a title/menu screen/view if the user explicitly requests one.

## Common Architecture Pitfalls

- **Unwired runtime behavior bodies** — Creating a static runtime behavior body (e.g., ground, wall) without wiring it to other bodies via `runtime behavior.add.collider()` or `runtime behavior.add.overlap()` has no user workflow effect. Every boundary or obstacle needs explicit collision wiring to the domain components it should interact with. After creating any static body, immediately add the collider call.
- **Interactive elements blocked by overlapping display objects** — When building UI (buttons, menus), the topmost display object in the screen/view list receives pointer events. Never hide the interactive element behind a decorative layer. Either make the visual element itself interactive, or ensure nothing is rendered on top of the hit area.
- **Polish before user workflow** — Adding particles, screen shake, and transitions before the core loop works is a common time sink. Get input -> action -> fail condition -> scoring -> restart working first. Everything else is polish.
- **No cleanup on restart** — Forgetting to remove event listeners, destroy timers, and dispose resources in `shutdown()` causes ghost behavior, double-firing events, and memory leaks after restart.

## Pre-Ship Validation Checklist

Before considering a application complete, verify all items:

- [ ] **Core loop** — user can start, play, lose/win, and see the result
- [ ] **Restart** — Works cleanly 3x in a row with identical behavior
- [ ] **Mobile input** — Touch/tap/swipe/gyro works; 44px minimum tap targets
- [ ] **Desktop input** — Keyboard + mouse works
- [ ] **Responsive** — Canvas resizes correctly on window resize
- [ ] **Constants** — Zero hardcoded magic numbers in application logic
- [ ] **EventBus** — No direct cross-module imports for communication
- [ ] **Cleanup** — All listeners removed in shutdown, resources disposed
- [ ] **Mute toggle** — See `mute-button` rule
- [ ] **Delta-based** — All movement uses delta time, not frame count
- [ ] **Build** — `npm run build` succeeds with no errors
- [ ] **No errors** — No uncaught exceptions or console errors at runtime
