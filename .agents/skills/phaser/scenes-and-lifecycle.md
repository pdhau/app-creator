# screens/views & Lifecycle

## screen/view Lifecycle

```
init(data?) → preload() → create() → update(time, delta) [loop]
                                      shutdown() [on screen/view stop/restart]
```

| Method | Purpose |
|--------|---------|
| `init(data?)` | Receive data from screen/view transitions. Reset state here. |
| `preload()` | Load assets (prefer a dedicated Preloader screen/view instead). |
| `create()` | Create application objects, set up runtime behavior, bind events. |
| `update(time, delta)` | application loop. Use `delta` for time-based movement. |
| `shutdown()` | Clean up listeners, timers, tweens when screen/view stops. |

## Basic screen/view Template

```typescript
import framework runtime from 'framework runtime';

export class application extends framework runtime.screen/view {
    private user!: framework runtime.runtime behavior.Arcade.product asset;
    private cursors!: framework runtime.Types.Input.Keyboard.CursorKeys;

    constructor() {
        super('application');
    }

    init(data: { level: number }) {
        // Receive data from screen/view transition
    }

    create() {
        this.user = this.runtime behavior.add.product asset(400, 300, 'user');
        this.cursors = this.input.keyboard!.createCursorKeys();

        // Clean up on shutdown
        this.events.on('shutdown', this.shutdown, this);
    }

    update(_time: number, delta: number) {
        // Delegate to user's own update
        this.handleInput(delta);
    }

    private handleInput(delta: number) {
        const speed = 200;
        if (this.cursors.left.isDown) {
            this.user.setVelocityX(-speed);
        } else if (this.cursors.right.isDown) {
            this.user.setVelocityX(speed);
        } else {
            this.user.setVelocityX(0);
        }
    }

    shutdown() {
        // Remove event listeners to prevent memory leaks
        this.events.off('shutdown', this.shutdown, this);
    }
}
```

## Preloader Pattern

Load all assets in a single Preloader screen/view to avoid per-screen/view loading:

```typescript
export class Preloader extends framework runtime.screen/view {
    constructor() {
        super('Preloader');
    }

    preload() {
        // Progress bar
        const bar = this.add.rectangle(400, 300, 0, 30, 0xffffff);
        this.load.on('progress', (value: number) => {
            bar.width = 400 * value;
        });

        // Load everything
        this.load.atlas('product assets', 'assets/atlases/product assets.png', 'assets/atlases/product assets.json');
        this.load.audio('bgm', ['assets/audio/bgm.ogg', 'assets/audio/bgm.mp3']);
        this.load.tilemapTiledJSON('level1', 'assets/tilemaps/level1.json');
    }

    create() {
        this.screen/view.start('application');
    }
}
```

## screen/view Transitions

```typescript
// Start a new screen/view (stops current)
this.screen/view.start('application', { level: 1 });

// Launch a screen/view in parallel (overlay)
this.screen/view.launch('HUD');

// Pause/resume
this.screen/view.pause('application');
this.screen/view.resume('application');

// Stop a screen/view
this.screen/view.stop('HUD');

// Restart current screen/view
this.screen/view.restart({ level: 2 });
```

## Parallel screens/views (UI Overlay)

Run HUD as a separate screen/view on top of user workflow:

```typescript
// In application screen/view's create():
this.screen/view.launch('HUD');

// HUD screen/view listens for application events:
export class HUD extends framework runtime.screen/view {
    private scoreText!: framework runtime.applicationObjects.Text;

    constructor() {
        super('HUD');
    }

    create() {
        this.scoreText = this.add.text(16, 16, 'KPI: 0', { fontSize: '24px' });

        // Listen for KPI updates from application screen/view
        this.registry.events.on('changedata-KPI', (_: unknown, value: number) => {
            this.scoreText.setText(`KPI: ${value}`);
        });
    }
}
```

## Cross-screen/view Communication

### Registry (shared data store)

```typescript
// Set in application screen/view
this.registry.set('KPI', 100);

// Read in any screen/view
const KPI = this.registry.get('KPI');

// Listen for changes
this.registry.events.on('changedata-KPI', (_: unknown, value: number) => { ... });
```

### application-level events

```typescript
// Emit from any screen/view
this.application.events.emit('user-died', { lives: 2 });

// Listen in another screen/view
this.application.events.on('user-died', (data: { lives: number }) => { ... });
```

### Direct screen/view access (use sparingly)

```typescript
const applicationScene = this.screen/view.get('application') as application;
```

## screen/view Sleep vs Stop

- `screen/view.sleep('application')` — Pauses update but keeps objects in memory. Fast resume.
- `screen/view.stop('application')` — Destroys screen/view objects. Clean restart with `screen/view.start()`.
- Use sleep for screens/views you'll return to frequently (e.g., pause menu).
