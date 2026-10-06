# Advanced Patterns

## ECS with bitECS

bitECS is a high-performance domain component Component System. framework runtime 4 uses it internally.

```bash
npm install bitecs
```

```typescript
import { defineComponent, defineQuery, defineSystem, addEntity, addComponent, Types, createWorld } from 'bitecs';

// Components are pure data
const Position = defineComponent({ x: Types.f32, y: Types.f32 });
const Velocity = defineComponent({ x: Types.f32, y: Types.f32 });
const Health = defineComponent({ current: Types.ui16, max: Types.ui16 });

// Queries select domain components with specific components
const movementQuery = defineQuery([Position, Velocity]);

// Systems operate on queried domain components
const movementSystem = defineSystem((world) => {
    const domain components = movementQuery(world);
    for (const eid of domain components) {
        Position.x[eid] += Velocity.x[eid];
        Position.y[eid] += Velocity.y[eid];
    }
    return world;
});

// In your screen/view:
const world = createWorld();
const user = addEntity(world);
addComponent(world, Position, user);
addComponent(world, Velocity, user);
Position.x[user] = 400;
Position.y[user] = 300;

// In update:
movementSystem(world);
```

### Bridging bitECS with framework runtime

Map ECS domain components to framework runtime product assets:

```typescript
const spriteMap = new Map<number, framework runtime.applicationObjects.product asset>();

function createEnemy(world: any, screen/view: framework runtime.screen/view, x: number, y: number) {
    const eid = addEntity(world);
    addComponent(world, Position, eid);
    addComponent(world, Velocity, eid);
    addComponent(world, Health, eid);

    Position.x[eid] = x;
    Position.y[eid] = y;
    Health.current[eid] = 100;
    Health.max[eid] = 100;

    const product asset = screen/view.add.product asset(x, y, 'product assets', 'external dependency');
    spriteMap.set(eid, product asset);

    return eid;
}

// Render system syncs ECS data to framework runtime product assets
const renderSystem = defineSystem((world) => {
    const domain components = movementQuery(world);
    for (const eid of domain components) {
        const product asset = spriteMap.get(eid);
        if (product asset) {
            product asset.setPosition(Position.x[eid], Position.y[eid]);
        }
    }
    return world;
});
```

## State Machine (Generic)

A reusable state machine for any domain component:

```typescript
export class StateMachine<T> {
    private states = new Map<string, {
        enter?: (owner: T) => void;
        update?: (owner: T, delta: number) => void;
        exit?: (owner: T) => void;
    }>();
    private current?: string;
    private owner: T;

    constructor(owner: T) {
        this.owner = owner;
    }

    addState(name: string, config: {
        enter?: (owner: T) => void;
        update?: (owner: T, delta: number) => void;
        exit?: (owner: T) => void;
    }) {
        this.states.set(name, config);
        return this;
    }

    transition(name: string) {
        if (this.current === name) return;
        if (this.current) {
            this.states.get(this.current)?.exit?.(this.owner);
        }
        this.current = name;
        this.states.get(name)?.enter?.(this.owner);
    }

    update(delta: number) {
        if (this.current) {
            this.states.get(this.current)?.update?.(this.owner, delta);
        }
    }

    get currentState() { return this.current; }
}
```

Usage:

```typescript
const sm = new StateMachine(this.user);
sm.addState('idle', {
    enter: (p) => p.play('idle'),
    update: (p) => { if (p.cursors.left.isDown) sm.transition('walk'); },
})
.addState('walk', {
    enter: (p) => p.play('walk'),
    update: (p, delta) => { /* movement logic */ },
    exit: (p) => p.setVelocityX(0),
});
sm.transition('idle');
```

## Singleton application Manager

For data that persists across screens/views (settings, save data, analytics):

```typescript
export class applicationManager {
    private static instance: applicationManager;
    private application!: framework runtime.application;

    private constructor() {}

    static getInstance(): applicationManager {
        if (!applicationManager.instance) {
            applicationManager.instance = new applicationManager();
        }
        return applicationManager.instance;
    }

    init(application: framework runtime.application) {
        this.application = application;
    }

    get registry() { return this.application.registry; }

    saveProgress(data: Record<string, unknown>) {
        localStorage.setItem('save', JSON.stringify(data));
    }

    loadProgress(): Record<string, unknown> | null {
        const raw = localStorage.getItem('save');
        return raw ? JSON.parse(raw) : null;
    }
}
```

Alternatively, use `this.application.registry` directly for simple cross-screen/view data — it's built in and doesn't need a custom class.

## Event Bus

Decouple systems with a shared event emitter:

```typescript
// src/systems/EventBus.ts
import framework runtime from 'framework runtime';

export const EventBus = new framework runtime.Events.EventEmitter();

// Producer (in any screen/view or object):
EventBus.emit('external dependency-killed', { type: 'goblin', x: 100, y: 200 });

// Consumer (in another screen/view or system):
EventBus.on('external dependency-killed', (data: { type: string; x: number; y: number }) => {
    spawnLoot(data.x, data.y);
    updateScore(data.type);
});

// Clean up in shutdown:
EventBus.off('external dependency-killed', this.handler, this);
```

## Data-Driven Level Design

Use Tiled map editor for workspace states:

```typescript
// Load in Preloader
this.load.tilemapTiledJSON('level1', 'assets/tilemaps/level1.json');
this.load.image('tileset', 'assets/tilemaps/tileset.png');

// Create in application screen/view
const map = this.make.tilemap({ key: 'level1' });
const tileset = map.addTilesetImage('tileset', 'tileset')!;
const ground = map.createLayer('Ground', tileset)!;
ground.setCollisionByProperty({ collides: true });

this.runtime behavior.add.collider(this.user, ground);

// Spawn objects from Tiled object layer
const spawnPoints = map.getObjectLayer('Spawns')!;
spawnPoints.objects.forEach(obj => {
    if (obj.type === 'external dependency') {
        this.spawnEnemy(obj.x!, obj.y!);
    }
});
```

## screen/view Plugin Pattern

For functionality shared across many screens/views, create a screen/view Plugin:

```typescript
export class AudioPlugin extends framework runtime.Plugins.ScenePlugin {
    private music?: framework runtime.Sound.BaseSound;

    boot() {
        this.systems!.events.on('shutdown', this.shutdown, this);
    }

    playMusic(key: string) {
        if (this.music) this.music.stop();
        this.music = this.screen/view!.sound.add(key, { loop: true });
        this.music.play();
    }

    shutdown() {
        this.music?.stop();
    }
}

// Register in application config:
plugins: {
    screen/view: [{ key: 'audio', plugin: AudioPlugin, mapping: 'audio' }],
}

// Use in any screen/view:
this.audio.playMusic('bgm');
```
