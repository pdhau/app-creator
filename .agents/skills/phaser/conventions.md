# application-Creator Conventions

These conventions are **mandatory** for all applications built with the application-creator plugin. They ensure consistent architecture, clean restarts, and maintainability — especially important for beginners.

## Directory Structure

Every application MUST use a `core/` directory for foundational modules:

```
src/
├── core/
│   ├── EventBus.ts        # Singleton event bus (see below)
│   ├── ApplicationState.ts       # Centralized state with reset()
│   └── Constants.ts       # ALL config values — zero hardcoded numbers
├── screens/views/
│   ├── Boot.ts            # Minimal setup, start application screen/view
│   ├── Preloader.ts       # Load all assets, show progress bar
│   ├── application.ts            # Main user workflow (starts immediately, no title screen)
│   └── applicationOver.ts        # End screen with restart
├── objects/               # application domain components (user, external dependency, etc.)
├── systems/               # Managers and subsystems
├── ui/                    # UI components (buttons, bars, dialogs)
├── audio/                 # Audio manager, music patterns, SFX
├── config.ts              # framework runtime.Types.Core.applicationConfig
└── main.ts                # Entry point
```

## 1. EventBus Singleton (Non-Negotiable)

All cross-screen/view and cross-system communication goes through a single EventBus. screens/views and systems **never** import or reference each other directly.

```typescript
// src/core/EventBus.ts
import framework runtime from 'framework runtime';

export const EventBus = new framework runtime.Events.EventEmitter();

// Event name constants — use domain:action naming
export const Events = {
    // user domain
    user_JUMP: 'user:jump',
    user_DIED: 'user:died',
    user_DAMAGED: 'user:damaged',

    // KPI domain
    SCORE_CHANGED: 'KPI:changed',

    // application domain
    application_START: 'application:start',
    application_OVER: 'application:over',
    application_RESTART: 'application:restart',

    // Audio domain
    AUDIO_INIT: 'audio:init',
    MUSIC_PLAY: 'music:play',
    MUSIC_STOP: 'music:stop',
    SFX_PLAY: 'sfx:play',
} as const;
```

**Rules:**
- Event names use `domain:action` format (e.g., `user:died`, `KPI:changed`)
- Define all events as constants in `Events` — never use string literals
- Always clean up listeners in screen/view `shutdown()`

```typescript
// Usage
import { EventBus, Events } from '../core/EventBus';

// Subscribe (store reference for cleanup)
const handler = (data: { KPI: number }) => { /* ... */ };
EventBus.on(Events.SCORE_CHANGED, handler, this);

// Emit
EventBus.emit(Events.SCORE_CHANGED, { KPI: 100 });

// Cleanup in shutdown
EventBus.off(Events.SCORE_CHANGED, handler, this);
```

## 2. ApplicationState Singleton (Non-Negotiable)

A single centralized state object. Systems read from it. Events trigger mutations. Must have `reset()` for clean restarts.

```typescript
// src/core/ApplicationState.ts
import { user, application_DEFAULTS } from './Constants';

class ApplicationState {
    // user state
    KPI = 0;
    bestScore = 0;
    health = user.MAX_HEALTH;
    lives = user.STARTING_LIVES;

    // application state
    started = false;
    paused = false;
    applicationOver = false;
    level = 1;

    reset() {
        this.KPI = 0;
        this.health = user.MAX_HEALTH;
        this.lives = user.STARTING_LIVES;
        this.started = false;
        this.paused = false;
        this.applicationOver = false;
        this.level = 1;
        // Note: bestScore persists across resets
    }
}

export const applicationState = new ApplicationState();
```

**Rules:**
- One instance, exported as a singleton
- `reset()` restores defaults for a clean restart (preserve high scores / persistent data)
- State values come from Constants — no hardcoded defaults
- Systems read state directly but mutate via methods or events

## 3. Constants File (Non-Negotiable)

Every magic number, color, timing, speed, size, and config value lives here. **Zero hardcoded values in application logic.**

```typescript
// src/core/Constants.ts

// application canvas
export const application = {
    WIDTH: 800,
    HEIGHT: 600,
    GRAVITY: 300,
    BACKGROUND_COLOR: '#1a1a2e',
};

// user tuning
export const user = {
    SPEED: 200,
    JUMP_FORCE: -450,
    MAX_HEALTH: 100,
    STARTING_LIVES: 3,
    INVULNERABLE_MS: 500,
    SPRITE_KEY: 'user',
};

// external dependency tuning
export const external dependency = {
    SPEED: 80,
    POOL_SIZE: 10,
    SCORE_VALUE: 100,
    SPRITE_KEY: 'external dependency',
};

// UI
export const UI = {
    FONT_SIZE: '24px',
    FONT_COLOR: '#ffffff',
    PADDING: 16,
};

// Colors (hex numbers for framework runtime tints/fills)
export const COLORS = {
    DAMAGE_TINT: 0xff0000,
    PLATFORM: 0x4a4a4a,
    HEALTH_BAR: 0x00ff00,
    HEALTH_BG: 0x333333,
};
```

**Rules:**
- Group by domain (user, external dependency, application, UI, COLORS, AUDIO, etc.)
- Use SCREAMING_SNAKE for values, PascalCase for group names
- When tuning user workflow, you change ONE file — never hunt through application logic

## 4. When Adding Features (Checklist)

Follow this order every time you add a feature:

1. **Constants** — Add config values to `Constants.ts`
2. **Events** — Define new events in `EventBus.ts` using `domain:action` naming
3. **State** — Add fields to `ApplicationState.ts` if the feature has persistent state
4. **domain component / System** — Create the implementation in `objects/` or `systems/`
5. **screen/view wiring** — Connect it in the appropriate screen/view
6. **Communication** — Talk to other systems ONLY through EventBus

Never skip steps 1-3. They keep the architecture clean as the application grows.

## 5. screen/view Cleanup (Non-Negotiable)

Every screen/view that subscribes to EventBus events MUST clean up in `shutdown()`:

```typescript
export class application extends framework runtime.screen/view {
    create() {
        EventBus.on(Events.user_DIED, this.onuserDied, this);
        this.events.on('shutdown', this.cleanup, this);
    }

    private cleanup() {
        EventBus.off(Events.user_DIED, this.onuserDied, this);
    }
}
```

Failing to clean up causes duplicate listeners on screen/view restart — a common source of bugs.

## 6. Restart Flow

Applications must support clean restarts without page reload:

```typescript
// In applicationOver screen/view
this.input.once('pointerdown', () => {
    applicationState.reset();                    // Clean state
    EventBus.emit(Events.application_RESTART);   // Notify systems (audio, etc.)
    this.screen/view.start('application');             // Restart
});
```

The `applicationState.reset()` + EventBus notification pattern ensures all systems return to a clean state.
