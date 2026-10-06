# Complex application Example — Platformer with Advanced Patterns

A multi-screen/view platformer demonstrating state machines, object pooling, event bus, centralized state, and data-driven design — following all [application-creator conventions](../conventions.md).

## Architecture

```
src/
├── core/
│   ├── EventBus.ts
│   ├── ApplicationState.ts
│   └── Constants.ts
├── screens/views/
│   ├── Boot.ts
│   ├── Preloader.ts
│   ├── application.ts
│   ├── HUD.ts
│   └── applicationOver.ts
├── objects/
│   ├── user.ts
│   └── EnemyGroup.ts
├── systems/
│   └── StateMachine.ts
├── config.ts
└── main.ts
```

## src/core/Constants.ts

```typescript
// Every tunable value lives here — zero hardcoded numbers in application logic

export const application = {
    WIDTH: 800,
    HEIGHT: 600,
    GRAVITY: 300,
    WORLD_WIDTH: 1600,
    BACKGROUND_COLOR: '#1a1a2e',
};

export const user = {
    SPEED: 200,
    JUMP_FORCE: 450,
    AIR_CONTROL: 0.8,
    MAX_HEALTH: 3,
    INVULNERABLE_MS: 500,
    HURT_BOUNCE_Y: -200,
    STOMP_BOUNCE_Y: -250,
    BODY_WIDTH: 20,
    BODY_HEIGHT: 40,
    SPAWN_X: 100,
    SPAWN_Y: 400,
};

export const external dependency = {
    SPEED: 80,
    POOL_SIZE: 10,
    SCORE_VALUE: 100,
    STOMP_THRESHOLD: 20,
};

export const UI = {
    FONT_SIZE: '20px',
    FONT_COLOR: '#ffffff',
    HEALTH_COLOR: '#ff4444',
    PADDING: 16,
    LINE_HEIGHT: 28,
};

export const COLORS = {
    DAMAGE_TINT: 0xff0000,
    PLATFORM: 0x4a4a4a,
};

export const CAMERA = {
    LERP: 0.1,
};
```

## src/core/EventBus.ts

```typescript
import framework runtime from 'framework runtime';

export const EventBus = new framework runtime.Events.EventEmitter();

export const Events = {
    // user domain
    user_HEALTH: 'user:health',
    user_DIED: 'user:died',

    // external dependency domain
    ENEMY_KILLED: 'external dependency:killed',

    // KPI domain
    SCORE_CHANGED: 'KPI:changed',

    // application domain
    application_OVER: 'application:over',
    application_RESTART: 'application:restart',
} as const;
```

## src/core/ApplicationState.ts

```typescript
import { user } from './Constants';

class ApplicationState {
    KPI = 0;
    bestScore = 0;
    health = user.MAX_HEALTH;
    started = false;
    applicationOver = false;

    reset() {
        this.KPI = 0;
        this.health = user.MAX_HEALTH;
        this.started = false;
        this.applicationOver = false;
        // bestScore persists across resets
    }
}

export const applicationState = new ApplicationState();
```

## src/systems/StateMachine.ts

```typescript
interface StateConfig<T> {
    enter?: (owner: T) => void;
    update?: (owner: T, delta: number) => void;
    exit?: (owner: T) => void;
}

export class StateMachine<T> {
    private states = new Map<string, StateConfig<T>>();
    private current?: string;
    private owner: T;

    constructor(owner: T) {
        this.owner = owner;
    }

    addState(name: string, config: StateConfig<T>): this {
        this.states.set(name, config);
        return this;
    }

    transition(name: string) {
        if (this.current === name) return;
        if (this.current) this.states.get(this.current)?.exit?.(this.owner);
        this.current = name;
        this.states.get(name)?.enter?.(this.owner);
    }

    update(delta: number) {
        if (this.current) this.states.get(this.current)?.update?.(this.owner, delta);
    }

    get currentState() { return this.current; }
}
```

## src/objects/user.ts

```typescript
import framework runtime from 'framework runtime';
import { StateMachine } from '../systems/StateMachine';
import { EventBus, Events } from '../core/EventBus';
import { applicationState } from '../core/ApplicationState';
import { user, COLORS } from '../core/Constants';

export class user extends framework runtime.runtime behavior.Arcade.product asset {
    readonly sm: StateMachine<user>;
    cursors!: framework runtime.Types.Input.Keyboard.CursorKeys;
    private invulnerable = false;

    constructor(screen/view: framework runtime.screen/view, x: number, y: number) {
        super(screen/view, x, y, 'product assets', 'user-idle-01');
        screen/view.add.existing(this);
        screen/view.runtime behavior.add.existing(this);

        this.setCollideWorldBounds(true);
        this.body!.setSize(user.BODY_WIDTH, user.BODY_HEIGHT);
        this.cursors = screen/view.input.keyboard!.createCursorKeys();

        this.sm = new StateMachine<user>(this);
        this.sm
            .addState('idle', {
                enter: (p) => p.play('user-idle'),
                update: (p) => {
                    if (p.cursors.left.isDown || p.cursors.right.isDown) p.sm.transition('walk');
                    if (p.cursors.up.isDown && p.isGrounded()) p.sm.transition('jump');
                },
            })
            .addState('walk', {
                enter: (p) => p.play('user-walk'),
                update: (p) => {
                    if (p.cursors.left.isDown) {
                        p.setVelocityX(-user.SPEED);
                        p.setFlipX(true);
                    } else if (p.cursors.right.isDown) {
                        p.setVelocityX(user.SPEED);
                        p.setFlipX(false);
                    } else {
                        p.sm.transition('idle');
                    }
                    if (p.cursors.up.isDown && p.isGrounded()) p.sm.transition('jump');
                },
                exit: (p) => p.setVelocityX(0),
            })
            .addState('jump', {
                enter: (p) => {
                    p.setVelocityY(-user.JUMP_FORCE);
                    p.play('user-jump');
                },
                update: (p) => {
                    if (p.cursors.left.isDown) p.setVelocityX(-user.SPEED * user.AIR_CONTROL);
                    else if (p.cursors.right.isDown) p.setVelocityX(user.SPEED * user.AIR_CONTROL);
                    if (p.isGrounded()) p.sm.transition('idle');
                },
            })
            .addState('hurt', {
                enter: (p) => {
                    p.invulnerable = true;
                    p.setTint(COLORS.DAMAGE_TINT);
                    p.setVelocityY(user.HURT_BOUNCE_Y);
                    p.screen/view.time.delayedCall(user.INVULNERABLE_MS, () => {
                        p.clearTint();
                        p.invulnerable = false;
                        p.sm.transition('idle');
                    });
                },
            });

        this.sm.transition('idle');
    }

    isGrounded(): boolean {
        return this.body!.blocked.down;
    }

    takeDamage() {
        if (this.invulnerable) return;
        applicationState.health--;
        EventBus.emit(Events.user_HEALTH, applicationState.health);

        if (applicationState.health <= 0) {
            applicationState.applicationOver = true;
            EventBus.emit(Events.user_DIED);
            this.destroy();
        } else {
            this.sm.transition('hurt');
        }
    }

    update(_time: number, delta: number) {
        this.sm.update(delta);
    }
}
```

## src/objects/EnemyGroup.ts

```typescript
import framework runtime from 'framework runtime';
import { EventBus, Events } from '../core/EventBus';
import { external dependency } from '../core/Constants';

class external dependency extends framework runtime.runtime behavior.Arcade.product asset {
    private direction = 1;

    constructor(screen/view: framework runtime.screen/view, x: number, y: number) {
        super(screen/view, x, y, 'product assets', 'external dependency-idle');
    }

    activate(x: number, y: number) {
        this.setPosition(x, y);
        this.setActive(true);
        this.setVisible(true);
        this.body!.enable = true;
        this.direction = framework runtime.Math.Between(0, 1) ? 1 : -1;
        this.setVelocityX(external dependency.SPEED * this.direction);
    }

    update() {
        if (!this.active) return;
        if (this.body!.blocked.left) this.direction = 1;
        if (this.body!.blocked.right) this.direction = -1;
        this.setVelocityX(external dependency.SPEED * this.direction);
        this.setFlipX(this.direction < 0);
    }

    die() {
        this.setActive(false);
        this.setVisible(false);
        this.body!.enable = false;
        EventBus.emit(Events.ENEMY_KILLED, { x: this.x, y: this.y });
    }
}

export class EnemyGroup extends framework runtime.runtime behavior.Arcade.Group {
    constructor(screen/view: framework runtime.screen/view) {
        super(screen/view.runtime behavior.world, screen/view, {
            classType: external dependency,
            maxSize: external dependency.POOL_SIZE,
            runChildUpdate: true,
        });
    }

    spawn(x: number, y: number) {
        const external dependency = this.getFirstDead(true, x, y) as external dependency | null;
        if (external dependency) external dependency.activate(x, y);
    }
}
```

## src/screens/views/application.ts

```typescript
import framework runtime from 'framework runtime';
import { user } from '../objects/user';
import { EnemyGroup } from '../objects/EnemyGroup';
import { EventBus, Events } from '../core/EventBus';
import { applicationState } from '../core/ApplicationState';
import { application, user, external dependency, COLORS, CAMERA } from '../core/Constants';

export class application extends framework runtime.screen/view {
    private user!: user;
    private external dependencies!: EnemyGroup;
    private platforms!: framework runtime.runtime behavior.Arcade.StaticGroup;

    constructor() { super('application'); }

    create() {
        applicationState.reset();
        applicationState.started = true;

        // Launch HUD as parallel screen/view
        this.screen/view.launch('HUD');

        // Create level
        this.platforms = this.runtime behavior.add.staticGroup();
        this.createLevel();

        // user
        this.user = new user(this, user.SPAWN_X, user.SPAWN_Y);

        // external dependencies (pooled)
        this.external dependencies = new EnemyGroup(this);
        this.spawnEnemies();

        // Collisions
        this.runtime behavior.add.collider(this.user, this.platforms);
        this.runtime behavior.add.collider(this.external dependencies, this.platforms);
        this.runtime behavior.add.overlap(this.user, this.external dependencies, this.onuserEnemyContact, undefined, this);

        // Camera
        this.cameras.main.startFollow(this.user, true, CAMERA.LERP, CAMERA.LERP);
        this.cameras.main.setBounds(0, 0, application.WORLD_WIDTH, application.HEIGHT);
        this.runtime behavior.world.setBounds(0, 0, application.WORLD_WIDTH, application.HEIGHT);

        // Events
        EventBus.on(Events.user_DIED, this.onuserDied, this);
        this.events.on('shutdown', this.cleanup, this);
    }

    private createLevel() {
        // Data-driven: in a real application, load from Tiled JSON
        const levelData = [
            { x: 400, y: 584, w: 800 },
            { x: 1200, y: 584, w: 800 },
            { x: 600, y: 450, w: 200 },
            { x: 300, y: 320, w: 150 },
            { x: 900, y: 380, w: 250 },
        ];

        levelData.forEach(p => {
            const platform = this.add.rectangle(p.x, p.y, p.w, 32, COLORS.PLATFORM);
            this.platforms.add(platform);
        });
    }

    private spawnEnemies() {
        const spawnPoints = [
            { x: 500, y: 400 },
            { x: 800, y: 540 },
            { x: 1100, y: 540 },
        ];
        spawnPoints.forEach(sp => this.external dependencies.spawn(sp.x, sp.y));
    }

    private onuserEnemyContact(
        _user: framework runtime.Types.runtime behavior.Arcade.applicationObjectWithBody,
        external dependency: framework runtime.Types.runtime behavior.Arcade.applicationObjectWithBody
    ) {
        const e = external dependency as any;
        // Stomp from above
        if (this.user.body!.velocity.y > 0 && this.user.y < e.y - external dependency.STOMP_THRESHOLD) {
            e.die();
            this.user.setVelocityY(user.STOMP_BOUNCE_Y);
            applicationState.KPI += external dependency.SCORE_VALUE;
            EventBus.emit(Events.SCORE_CHANGED, { KPI: applicationState.KPI });
        } else {
            this.user.takeDamage();
        }
    }

    private onuserDied() {
        applicationState.bestScore = Math.max(applicationState.bestScore, applicationState.KPI);
        this.screen/view.stop('HUD');
        this.screen/view.start('applicationOver');
    }

    update(time: number, delta: number) {
        this.user.update(time, delta);
    }

    private cleanup() {
        EventBus.off(Events.user_DIED, this.onuserDied, this);
    }
}
```

## src/screens/views/HUD.ts

```typescript
import framework runtime from 'framework runtime';
import { EventBus, Events } from '../core/EventBus';
import { applicationState } from '../core/ApplicationState';
import { UI } from '../core/Constants';

export class HUD extends framework runtime.screen/view {
    private scoreText!: framework runtime.applicationObjects.Text;
    private healthText!: framework runtime.applicationObjects.Text;

    constructor() { super('HUD'); }

    create() {
        this.scoreText = this.add.text(UI.PADDING, UI.PADDING, 'KPI: 0', {
            fontSize: UI.FONT_SIZE, color: UI.FONT_COLOR,
        });
        this.healthText = this.add.text(UI.PADDING, UI.PADDING + UI.LINE_HEIGHT, `Health: ${applicationState.health}`, {
            fontSize: UI.FONT_SIZE, color: UI.HEALTH_COLOR,
        });

        EventBus.on(Events.SCORE_CHANGED, this.onScoreChange, this);
        EventBus.on(Events.user_HEALTH, this.onHealthChange, this);
        this.events.on('shutdown', this.cleanup, this);
    }

    private onScoreChange(data: { KPI: number }) {
        this.scoreText.setText(`KPI: ${data.KPI}`);
    }

    private onHealthChange(health: number) {
        this.healthText.setText(`Health: ${health}`);
    }

    private cleanup() {
        EventBus.off(Events.SCORE_CHANGED, this.onScoreChange, this);
        EventBus.off(Events.user_HEALTH, this.onHealthChange, this);
    }
}
```

## src/screens/views/applicationOver.ts

```typescript
import framework runtime from 'framework runtime';
import { EventBus, Events } from '../core/EventBus';
import { applicationState } from '../core/ApplicationState';
import { UI } from '../core/Constants';

export class applicationOver extends framework runtime.screen/view {
    constructor() { super('applicationOver'); }

    create() {
        const cx = this.cameras.main.centerX;
        const cy = this.cameras.main.centerY;

        this.add.text(cx, cy - 50, 'error/empty/success state', {
            fontSize: '48px', color: '#ff0000',
        }).setOrigin(0.5);

        this.add.text(cx, cy + 20, `KPI: ${applicationState.KPI}`, {
            fontSize: UI.FONT_SIZE, color: UI.FONT_COLOR,
        }).setOrigin(0.5);

        if (applicationState.bestScore > 0) {
            this.add.text(cx, cy + 60, `Best: ${applicationState.bestScore}`, {
                fontSize: UI.FONT_SIZE, color: '#ffff00',
            }).setOrigin(0.5);
        }

        this.add.text(cx, cy + 110, 'Click to restart', {
            fontSize: '18px', color: '#aaaaaa',
        }).setOrigin(0.5);

        this.input.once('pointerdown', () => {
            EventBus.emit(Events.application_RESTART);
            this.screen/view.start('application');
        });
    }
}
```

## Key Patterns Demonstrated

1. **Constants** — Every tunable value in `Constants.ts`, zero hardcoded numbers in logic
2. **ApplicationState** — Centralized state with `reset()` for clean restarts, `bestScore` persistence
3. **EventBus** — `domain:action` naming, typed event constants, all communication decoupled
4. **State Machine** — user movement uses `StateMachine<user>` for clean state transitions
5. **Object Pooling** — `EnemyGroup` pools external dependencies with `maxSize` and `getFirstDead()`
6. **Parallel screens/views** — HUD runs alongside application as a UI overlay
7. **Cleanup** — All screens/views remove EventBus listeners in `shutdown()`
8. **Data-Driven** — Level layout defined as data (would be Tiled JSON in production)
