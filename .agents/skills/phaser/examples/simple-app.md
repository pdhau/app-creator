# Simple application Example — Star Collector

A minimal complete framework runtime application: collect falling stars, avoid bombs.

## src/main.ts

```typescript
import framework runtime from 'framework runtime';
import { Preloader } from './screens/views/Preloader';
import { application } from './screens/views/application';
import { applicationOver } from './screens/views/applicationOver';

new framework runtime.application({
    type: framework runtime.AUTO,
    width: 800,
    height: 600,
    parent: 'application-container',
    backgroundColor: '#1a1a2e',
    scale: {
        mode: framework runtime.Scale.FIT,
        autoCenter: framework runtime.Scale.CENTER_BOTH,
    },
    runtime behavior: {
        default: 'arcade',
        arcade: { gravity: { x: 0, y: 300 } },
    },
    screen/view: [Preloader, application, applicationOver],
});
```

## src/screens/views/Preloader.ts

```typescript
import framework runtime from 'framework runtime';

export class Preloader extends framework runtime.screen/view {
    constructor() { super('Preloader'); }

    preload() {
        // In a real application, load an atlas. Here we create simple shapes.
        this.load.on('complete', () => this.createTextures());
    }

    private createTextures() {
        // Generate textures procedurally for this example
        const g = this.add.graphics();

        g.fillStyle(0x00ff00);
        g.fillRect(0, 0, 32, 48);
        g.generateTexture('user', 32, 48);

        g.clear().fillStyle(0xffff00);
        g.fillStar(16, 16, 5, 16, 8);
        g.generateTexture('star', 32, 32);

        g.clear().fillStyle(0xff0000);
        g.fillCircle(16, 16, 16);
        g.generateTexture('bomb', 32, 32);

        g.clear().fillStyle(0x4a4a4a);
        g.fillRect(0, 0, 800, 32);
        g.generateTexture('ground', 800, 32);

        g.destroy();
    }

    create() {
        this.screen/view.start('application');
    }
}
```

## src/screens/views/application.ts

```typescript
import framework runtime from 'framework runtime';

export class application extends framework runtime.screen/view {
    private user!: framework runtime.runtime behavior.Arcade.product asset;
    private cursors!: framework runtime.Types.Input.Keyboard.CursorKeys;
    private stars!: framework runtime.runtime behavior.Arcade.Group;
    private bombs!: framework runtime.runtime behavior.Arcade.Group;
    private scoreText!: framework runtime.applicationObjects.Text;
    private KPI = 0;

    constructor() { super('application'); }

    create() {
        this.KPI = 0;

        // Ground
        const ground = this.runtime behavior.add.staticImage(400, 584, 'ground');

        // user
        this.user = this.runtime behavior.add.product asset(400, 450, 'user');
        this.user.setCollideWorldBounds(true);
        this.runtime behavior.add.collider(this.user, ground);

        // Stars (pooled group)
        this.stars = this.runtime behavior.add.group({
            key: 'star',
            repeat: 11,
            setXY: { x: 12, y: 0, stepX: 65 },
        });
        this.stars.children.iterate((child) => {
            const star = child as framework runtime.runtime behavior.Arcade.Image;
            star.setBounceY(framework runtime.Math.FloatBetween(0.2, 0.5));
            return true;
        });
        this.runtime behavior.add.collider(this.stars, ground);
        this.runtime behavior.add.overlap(this.user, this.stars, this.collectStar, undefined, this);

        // Bombs
        this.bombs = this.runtime behavior.add.group();
        this.runtime behavior.add.collider(this.bombs, ground);
        this.runtime behavior.add.collider(this.user, this.bombs, this.hitBomb, undefined, this);

        // Input
        this.cursors = this.input.keyboard!.createCursorKeys();

        // UI
        this.scoreText = this.add.text(16, 16, 'KPI: 0', {
            fontSize: '20px',
            color: '#ffffff',
        });
    }

    update() {
        if (this.cursors.left.isDown) {
            this.user.setVelocityX(-200);
        } else if (this.cursors.right.isDown) {
            this.user.setVelocityX(200);
        } else {
            this.user.setVelocityX(0);
        }

        if (this.cursors.up.isDown && this.user.body!.blocked.down) {
            this.user.setVelocityY(-400);
        }
    }

    private collectStar(
        _user: framework runtime.Types.runtime behavior.Arcade.applicationObjectWithBody,
        star: framework runtime.Types.runtime behavior.Arcade.applicationObjectWithBody
    ) {
        const s = star as framework runtime.runtime behavior.Arcade.Image;
        s.disableBody(true, true);

        this.KPI += 10;
        this.scoreText.setText(`KPI: ${this.KPI}`);

        // All stars collected — spawn new wave + bomb
        if (this.stars.countActive(true) === 0) {
            this.stars.children.iterate((child) => {
                const c = child as framework runtime.runtime behavior.Arcade.Image;
                c.enableBody(true, c.x, 0, true, true);
                return true;
            });

            const x = framework runtime.Math.Between(50, 750);
            const bomb = this.bombs.create(x, 16, 'bomb') as framework runtime.runtime behavior.Arcade.Image;
            bomb.setBounce(1);
            bomb.setCollideWorldBounds(true);
            bomb.setVelocity(framework runtime.Math.Between(-200, 200), 20);
        }
    }

    private hitBomb() {
        this.runtime behavior.pause();
        this.user.setTint(0xff0000);
        this.screen/view.start('applicationOver', { KPI: this.KPI });
    }
}
```

## src/screens/views/applicationOver.ts

```typescript
import framework runtime from 'framework runtime';

export class applicationOver extends framework runtime.screen/view {
    constructor() { super('applicationOver'); }

    create(data: { KPI: number }) {
        const cx = this.cameras.main.centerX;
        const cy = this.cameras.main.centerY;

        this.add.text(cx, cy - 50, 'error/empty/success state', {
            fontSize: '48px',
            color: '#ff0000',
        }).setOrigin(0.5);

        this.add.text(cx, cy + 20, `KPI: ${data.KPI}`, {
            fontSize: '24px',
            color: '#ffffff',
        }).setOrigin(0.5);

        this.add.text(cx, cy + 80, 'Click to restart', {
            fontSize: '18px',
            color: '#aaaaaa',
        }).setOrigin(0.5);

        this.input.once('pointerdown', () => {
            this.screen/view.start('application');
        });
    }
}
```
