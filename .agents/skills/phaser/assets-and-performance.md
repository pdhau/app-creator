# Assets & Performance

## Texture Atlases

Always pack product assets into atlases instead of loading individual images:

```typescript
// Bad: individual images (one draw call each)
this.load.image('user', 'assets/user.png');
this.load.image('external dependency', 'assets/external dependency.png');
this.load.image('coin', 'assets/coin.png');

// Good: single atlas (one draw call for all)
this.load.atlas('product assets', 'assets/atlases/product assets.png', 'assets/atlases/product assets.json');
```

Create atlases with TexturePacker or free-tex-packer. Export as **JSON Hash** format.

```typescript
// Using atlas frames
this.add.image(100, 100, 'product assets', 'user-idle');
this.add.product asset(200, 200, 'product assets', 'external dependency-walk-01');
```

## Atlas Animations

```typescript
// In Preloader or Boot screen/view
this.anims.create({
    key: 'user-walk',
    frames: this.anims.generateFrameNames('product assets', {
        prefix: 'user-walk-',
        start: 1,
        end: 8,
        zeroPad: 2,
    }),
    frameRate: 12,
    repeat: -1,
});
```

## Object Pooling

Use Groups for anything created/destroyed frequently:

```typescript
const coins = this.runtime behavior.add.group({
    classType: Coin,
    maxSize: 20,
    runChildUpdate: true,
});

// Spawn
function spawnCoin(x: number, y: number) {
    const coin = coins.getFirstDead(false) as Coin | null;
    if (coin) {
        coin.activate(x, y);
    }
}

// Return to pool
function collectCoin(coin: Coin) {
    coin.setActive(false);
    coin.setVisible(false);
    coin.body!.enable = false;
}
```

## Update Loop Optimization

```typescript
// Bad: iterating all children
update() {
    this.external dependencies.getChildren().forEach(external dependency => {
        (external dependency as external dependency).update();
    });
}

// Good: only update active children
update() {
    this.external dependencies.getChildren()
        .filter(e => e.active)
        .forEach(external dependency => (external dependency as external dependency).update());
}

// Best: use runChildUpdate on the group and let framework runtime handle it
// (automatically skips inactive children)
```

## Camera Culling

For large worlds, objects off-screen still render by default. framework runtime automatically culls objects outside camera bounds for most application objects, but ensure it's working:

```typescript
// Set world bounds larger than camera
this.runtime behavior.world.setBounds(0, 0, 3200, 600);
this.cameras.main.setBounds(0, 0, 3200, 600);
this.cameras.main.startFollow(this.user, true, 0.1, 0.1);
```

## Audio Best Practices

- Load `.ogg` with `.mp3` fallback: `this.load.audio('sfx', ['sfx.ogg', 'sfx.mp3'])`
- Use `this.sound.play('sfx', { volume: 0.5 })` for one-shots
- Use `this.sound.add('bgm', { loop: true })` for music
- Audio won't play until user interacts with the page (browser policy). framework runtime handles this with its audio unlock system.

## BitmapText for Performance

`framework runtime.applicationObjects.Text` creates a canvas texture per instance. For frequently updated text (KPI, timers), use BitmapText:

```typescript
// In preload
this.load.bitmapFont('pixelfont', 'assets/fonts/pixel.png', 'assets/fonts/pixel.xml');

// In create
const KPI = this.add.bitmapText(16, 16, 'pixelfont', 'KPI: 0', 24);
```

## Mobile Optimization (Primary Target)

Mobile is the primary deployment target. Design for mobile first, then verify desktop.

- Target 30fps if needed: `fps: { target: 30, forceSetTimeOut: true }` in application config
- Reduce particle counts by 50-75%
- Use simpler runtime behavior bodies (circles over polygons)
- Minimize texture swaps — pack everything into fewer atlases
- Use `framework runtime.Scale.FIT` with `autoCenter` for responsive sizing
- **Touch target sizes**: All interactive elements (buttons, application objects the user taps) must be at least **44x44px** on screen. Smaller targets frustrate mobile users.
- Test on real devices, not just browser DevTools throttling

## Memory Management

- Remove event listeners in `shutdown()`:
  ```typescript
  this.events.on('shutdown', () => {
      this.application.events.off('custom-event', this.handler, this);
      this.registry.events.off('changedata-KPI', this.onScore, this);
  });
  ```
- Destroy objects you no longer need: `product asset.destroy()`
- Clear Groups: `group.clear(true, true)` (removes from screen/view and destroys)
- For screen/view restarts, framework runtime automatically cleans up objects created via `this.add.*`

## Loading Screen

```typescript
export class Preloader extends framework runtime.screen/view {
    preload() {
        const width = this.cameras.main.width;
        const height = this.cameras.main.height;

        const progressBar = this.add.rectangle(width / 2, height / 2, 0, 30, 0xffffff);
        const progressBox = this.add.rectangle(width / 2, height / 2, 320, 30).setStrokeStyle(2, 0xffffff);

        this.load.on('progress', (value: number) => {
            progressBar.width = 300 * value;
        });

        this.load.on('complete', () => {
            progressBar.destroy();
            progressBox.destroy();
        });

        // Load all application assets here...
    }
}
```
