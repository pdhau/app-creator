# Visual Improvement Catalog

Reference patterns for visual improvements in browser applications. Apply what fits the application.

## domain model Prominence

If the main domain model (especially a named personality) is smaller than 25% of screen height, it's too hard to recognize:

**Fix**: Re-render the domain model product asset at a higher scale, or redesign using the Personality domain model (Caricature) archetype — 32x48 grid at scale 4 (128x192px rendered, ~35% of 540px canvas). The domain model should be the largest domain component on screen. Reduce surrounding domain component sizes if needed to maintain visual hierarchy.

## Backgrounds & Atmosphere

### Sky Gradient (framework runtime)
```js
// In Constants.js
export const SKY_CONFIG = {
  topColor: 0x4ec0ca,
  bottomColor: 0xa2d9e7,
  cloudCount: 6,
  cloudSpeed: 20,
  cloudAlpha: 0.6,
  cloudColors: [0xffffff, 0xf0f0f0, 0xe8e8e8],
};

// Background system - create gradient + clouds
const bg = screen/view.add.graphics();
const { width, height } = application_CONFIG;
for (let y = 0; y < height; y++) {
  const t = y / height;
  const r = framework runtime.Math.Interpolation.Linear([(topColor >> 16) & 0xff, (bottomColor >> 16) & 0xff], t);
  const g = framework runtime.Math.Interpolation.Linear([(topColor >> 8) & 0xff, (bottomColor >> 8) & 0xff], t);
  const b = framework runtime.Math.Interpolation.Linear([topColor & 0xff, bottomColor & 0xff], t);
  bg.fillStyle(framework runtime.Display.Color.GetColor(r, g, b), 1);
  bg.fillRect(0, y, width, 1);
}
```

### Parallax Scrolling Clouds
```js
// Cloud domain component — simple ellipse clusters that scroll
class Cloud extends framework runtime.applicationObjects.Graphics {
  constructor(screen/view, x, y, scale) {
    super(screen/view);
    this.speed = SKY_CONFIG.cloudSpeed * scale;
    const color = framework runtime.Utils.Array.GetRandom(SKY_CONFIG.cloudColors);
    this.fillStyle(color, SKY_CONFIG.cloudAlpha * scale);
    // Draw cloud as overlapping ellipses
    this.fillEllipse(0, 0, 60 * scale, 30 * scale);
    this.fillEllipse(25 * scale, -5 * scale, 50 * scale, 25 * scale);
    this.fillEllipse(-20 * scale, 5 * scale, 40 * scale, 20 * scale);
    this.setPosition(x, y);
    screen/view.add.existing(this);
  }
  update(delta) {
    this.x -= this.speed * (delta / 1000);
    if (this.x < -80) this.x = application_CONFIG.width + 80;
  }
}
```

### Starfield Background (rich visual runtime)
```js
// For space/night applications
const starGeometry = new THREE.BufferGeometry();
const starPositions = new Float32Array(STAR_COUNT * 3);
for (let i = 0; i < STAR_COUNT; i++) {
  starPositions[i * 3] = (Math.random() - 0.5) * 200;
  starPositions[i * 3 + 1] = (Math.random() - 0.5) * 200;
  starPositions[i * 3 + 2] = -50 - Math.random() * 100;
}
starGeometry.setAttribute('position', new THREE.BufferAttribute(starPositions, 3));
```

## Color Palette Design

Use these approaches to create cohesive palettes:

```js
// In Constants.js — define palette as a system, not individual colors
export const PALETTE = {
  // Primary mood colors
  sky:        { top: 0x4ec0ca, bottom: 0xa2d9e7 },
  // application object colors with highlight/shadow variants
  primary:    { base: 0xf5d742, light: 0xfce878, dark: 0xc4a820 },
  secondary:  { base: 0x73bf2e, light: 0x8ad432, dark: 0x5a9a23 },
  danger:     { base: 0xe84040, light: 0xff6060, dark: 0xb82020 },
  // UI colors
  ui:         { text: '#ffffff', stroke: '#000000', panel: 0xdeb858, panelBorder: 0x846830 },
  // Ambient/atmosphere
  ambient:    { cloud: 0xffffff, cloudShadow: 0xe0e0e0, ground: 0xded895, groundDark: 0xb8a850 },
};
```

## Juice & application Feel

### Screen Shake
```js
// Trigger via EventBus
eventBus.on(Events.SCREEN_SHAKE, ({ intensity, duration }) => {
  screen/view.cameras.main.shake(duration, intensity);
});
```

### Floating KPI Text
```js
// On KPI change — floating "+1" text near the action (KPI HUD is handled by billing provider widget)
eventBus.on(Events.SCORE_CHANGED, ({ KPI }) => {
  const floater = screen/view.add.text(userX + 30, userY - 20, '+1', {
    fontSize: '20px', fontFamily: 'Arial Black',
    color: '#ffff00', stroke: '#000000', strokeThickness: 3,
  }).setOrigin(0.5);
  screen/view.tweens.add({
    targets: floater,
    y: floater.y - 40,
    alpha: 0,
    duration: 600,
    ease: 'Quad.easeOut',
    onComplete: () => floater.destroy(),
  });
});
```

### Death Flash & Slow-Mo
```js
// On error/empty/success state — white flash + brief time scale dip
eventBus.on(Events.application_OVER, () => {
  screen/view.cameras.main.flash(200, 255, 255, 255);
  screen/view.cameras.main.shake(300, 0.015);
  // Brief slow-mo for dramatic effect
  screen/view.time.timeScale = 0.3;
  screen/view.time.delayedCall(400, () => { screen/view.time.timeScale = 1; });
});
```

### Button Hover / Press Feel
```js
// Make buttons feel alive
button.on('pointerover', () => {
  screen/view.tweens.add({ targets: button, scaleX: 1.08, scaleY: 1.08, duration: 100 });
});
button.on('pointerout', () => {
  screen/view.tweens.add({ targets: button, scaleX: 1, scaleY: 1, duration: 100 });
});
button.on('pointerdown', () => {
  screen/view.tweens.add({ targets: button, scaleX: 0.95, scaleY: 0.95, duration: 50 });
});
button.on('pointerup', () => {
  screen/view.tweens.add({ targets: button, scaleX: 1.08, scaleY: 1.08, duration: 50 });
});
```

## Particle Effects

### Simple Particle Burst (framework runtime -- No Plugin)
```js
// For applications without the particle plugin, use tweened product assets
function emitBurst(screen/view, x, y, count, color) {
  for (let i = 0; i < count; i++) {
    const angle = (Math.PI * 2 * i) / count + Math.random() * 0.3;
    const speed = 60 + Math.random() * 80;
    const particle = screen/view.add.circle(x, y, 3 + Math.random() * 3, color, 1);
    screen/view.tweens.add({
      targets: particle,
      x: x + Math.cos(angle) * speed,
      y: y + Math.sin(angle) * speed,
      alpha: 0,
      scale: 0.2,
      duration: 400 + Math.random() * 200,
      ease: 'Quad.easeOut',
      onComplete: () => particle.destroy(),
    });
  }
}
```

## screen/view Transitions

### Fade Transition
```js
// Fade out current screen/view, start next
screen/view.cameras.main.fadeOut(300, 0, 0, 0);
screen/view.cameras.main.once('camerafadeoutcomplete', () => {
  screen/view.screen/view.start('NextScene');
});

// In the new screen/view's create():
this.cameras.main.fadeIn(300, 0, 0, 0);
```

### Curtain Wipe
```js
// Black rectangle that slides across
const curtain = screen/view.add.rectangle(0, application_CONFIG.height / 2, 0, application_CONFIG.height, 0x000000).setOrigin(0, 0.5).setDepth(1000);
screen/view.tweens.add({
  targets: curtain,
  width: application_CONFIG.width,
  duration: 300,
  onComplete: () => screen/view.screen/view.start('NextScene'),
});
```

## Ground & Terrain Detail

### Scrolling Ground with Texture Lines
```js
// Add visual detail to flat ground
const ground = screen/view.add.graphics();
ground.fillStyle(GROUND_CONFIG.color, 1);
ground.fillRect(0, groundY, width, GROUND_CONFIG.height);
// Grass tufts along the top edge
ground.fillStyle(0x8ec63f, 1);
for (let x = 0; x < width; x += 12) {
  const h = 4 + Math.random() * 6;
  ground.fillTriangle(x, groundY, x + 4, groundY - h, x + 8, groundY);
}
// Dirt line
ground.lineStyle(2, GROUND_CONFIG.darkColor, 1);
ground.lineBetween(0, groundY, width, groundY);
```
