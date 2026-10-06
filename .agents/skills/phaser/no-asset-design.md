# No-Asset Visual Design

Techniques for making asset-free framework runtime applications look polished using only procedural graphics (`framework runtime.applicationObjects.Graphics`), shapes, particles, and tweens. No product asset sheets, no image files — just code.

## Color Palette

Pick a cohesive 3-5 color palette before writing any visual code. Define it in `Constants.ts`:

```typescript
// Use coolors.co or lospec.com/palette-list for inspiration
export const PALETTE = {
    SKY_TOP: 0x1a1a2e,
    SKY_BOTTOM: 0x16213e,
    ACCENT: 0xe94560,
    FOREGROUND: 0x0f3460,
    HIGHLIGHT: 0xf5d742,
    TEXT: '#ffffff',
} as const;
```

**Rules:**
- Every color in the application comes from the palette — no ad-hoc hex values in application logic
- Limit to 5 colors max. Constraint reads as intentional style, not placeholder art
- Use opacity variations (0.3, 0.5, 0.8) of palette colors for depth rather than adding new colors

## Gradient Backgrounds

Flat solid backgrounds look unfinished. Always use gradients:

```typescript
// Sky gradient — call once in create(), behind everything
private createSkyGradient(): void {
    const g = this.add.graphics();
    g.fillGradientStyle(
        PALETTE.SKY_TOP, PALETTE.SKY_TOP,
        PALETTE.SKY_BOTTOM, PALETTE.SKY_BOTTOM,
        1
    );
    g.fillRect(0, 0, application.WIDTH, application.HEIGHT);
    g.setDepth(-100);
}
```

### Time-of-Day Color Shifts

Lerp between palettes for dawn/dusk cycles:

```typescript
private lerpColor(a: number, b: number, t: number): number {
    const ar = (a >> 16) & 0xff, ag = (a >> 8) & 0xff, ab = a & 0xff;
    const br = (b >> 16) & 0xff, bg = (b >> 8) & 0xff, bb = b & 0xff;
    const r = Math.round(ar + (br - ar) * t);
    const g = Math.round(ag + (bg - ag) * t);
    const blue = Math.round(ab + (bb - ab) * t);
    return (r << 16) | (g << 8) | blue;
}
```

## Parallax Depth

3-4 layers minimum. Each layer moves at a different speed relative to the camera or user. Even subtle differences (0.1x, 0.3x, 0.6x, 1x) create massive depth perception.

```typescript
interface ParallaxLayer {
    elements: framework runtime.applicationObjects.applicationObject[];
    speed: number;
}

private layers: ParallaxLayer[] = [];

private createParallax(): void {
    // Far background — barely moves
    this.layers.push({
        elements: this.createCloudLayer(3, 0.15),
        speed: 0.1,
    });
    // Mid layer
    this.layers.push({
        elements: this.createHillLayer(0.4),
        speed: 0.3,
    });
    // Near layer — moves faster
    this.layers.push({
        elements: this.createCloudLayer(5, 0.5),
        speed: 0.6,
    });
}

update(_time: number, delta: number): void {
    for (const layer of this.layers) {
        for (const el of layer.elements) {
            (el as framework runtime.applicationObjects.Shape).x -= layer.speed * (delta / 16);
            // Wrap around
            if ((el as framework runtime.applicationObjects.Shape).x < -100) {
                (el as framework runtime.applicationObjects.Shape).x = application.WIDTH + framework runtime.Math.Between(50, 200);
            }
        }
    }
}
```

## Procedural Elements

### Clouds

Rounded rectangles or overlapping circles with slight transparency, random sizes, drifting slowly:

```typescript
private createCloud(x: number, y: number, scale: number, alpha: number): framework runtime.applicationObjects.Graphics {
    const g = this.add.graphics();
    g.fillStyle(0xffffff, alpha);

    // Overlapping circles for puffy cloud shape
    const baseR = 20 * scale;
    g.fillCircle(0, 0, baseR);
    g.fillCircle(-baseR * 0.7, baseR * 0.2, baseR * 0.7);
    g.fillCircle(baseR * 0.7, baseR * 0.2, baseR * 0.8);
    g.fillCircle(baseR * 0.3, -baseR * 0.3, baseR * 0.6);

    g.setPosition(x, y);
    return g;
}

private createCloudLayer(count: number, alphaBase: number): framework runtime.applicationObjects.Graphics[] {
    const clouds: framework runtime.applicationObjects.Graphics[] = [];
    for (let i = 0; i < count; i++) {
        const x = framework runtime.Math.Between(0, application.WIDTH);
        const y = framework runtime.Math.Between(30, application.HEIGHT * 0.4);
        const scale = framework runtime.Math.FloatBetween(0.5, 1.5);
        const alpha = alphaBase * framework runtime.Math.FloatBetween(0.5, 1.0);
        clouds.push(this.createCloud(x, y, scale, alpha));
    }
    return clouds;
}
```

### Mountains / Hills

Sine waves or noise drawn to Graphics, layered with different opacities:

```typescript
private createHills(color: number, alpha: number, amplitude: number, frequency: number, yOffset: number, depth: number): framework runtime.applicationObjects.Graphics {
    const g = this.add.graphics();
    g.fillStyle(color, alpha);
    g.beginPath();
    g.moveTo(0, application.HEIGHT);

    for (let x = 0; x <= application.WIDTH; x += 4) {
        const y = yOffset + Math.sin(x * frequency * 0.01) * amplitude
                           + Math.sin(x * frequency * 0.023) * (amplitude * 0.5);
        g.lineTo(x, y);
    }

    g.lineTo(application.WIDTH, application.HEIGHT);
    g.closePath();
    g.fillPath();
    g.setDepth(depth);
    return g;
}
```

### Ambient Particles

Use framework runtime's particle system for dust, leaves, light motes — ambient life without assets:

```typescript
private createAmbientParticles(): void {
    // Create a tiny circle texture for particles
    const g = this.add.graphics();
    g.fillStyle(0xffffff);
    g.fillCircle(4, 4, 4);
    g.generateTexture('particle', 8, 8);
    g.destroy();

    this.add.particles(0, 0, 'particle', {
        x: { min: 0, max: application.WIDTH },
        y: { min: 0, max: application.HEIGHT },
        scale: { start: 0.3, end: 0 },
        alpha: { start: 0.3, end: 0 },
        speed: { min: 5, max: 20 },
        lifespan: 4000,
        frequency: 500,
        blendMode: 'ADD',
        tint: PALETTE.HIGHLIGHT,
    });
}
```

## "Lighting" Without Shaders

### Vignette Overlay

Dark edges, transparent center — adds instant cinematic polish:

```typescript
private createVignette(): void {
    const g = this.add.graphics();
    const cx = application.WIDTH / 2;
    const cy = application.HEIGHT / 2;
    const r = Math.max(application.WIDTH, application.HEIGHT) * 0.7;

    // Radial gradient approximation with concentric rings
    const steps = 20;
    for (let i = steps; i >= 0; i--) {
        const t = i / steps;
        const alpha = (1 - t) * 0.6; // Max 0.6 opacity at edges
        g.fillStyle(0x000000, alpha);
        g.fillCircle(cx, cy, r * t + r * 0.3);
    }
    g.setDepth(1000); // Above everything
}
```

### user Glow

Subtle radial gradient following the user:

```typescript
private createGlow(target: framework runtime.applicationObjects.applicationObject, color: number, radius: number): framework runtime.applicationObjects.Graphics {
    const g = this.add.graphics();
    const steps = 10;
    for (let i = steps; i >= 0; i--) {
        const t = i / steps;
        g.fillStyle(color, t * 0.15);
        g.fillCircle(0, 0, radius * (1 - t * 0.5));
    }
    g.setBlendMode(framework runtime.BlendModes.ADD);
    return g;
}

// In update: glow.setPosition(user.x, user.y);
```

### Sun Rays

Additive blend mode on white/yellow shapes:

```typescript
private createSunRays(): void {
    const g = this.add.graphics();
    g.setBlendMode(framework runtime.BlendModes.ADD);

    for (let i = 0; i < 5; i++) {
        const angle = framework runtime.Math.DegToRad(-30 + i * 15);
        const length = application.HEIGHT * 1.5;
        g.fillStyle(0xffffff, 0.03);
        g.beginPath();
        g.moveTo(application.WIDTH * 0.8, 0);
        g.lineTo(
            application.WIDTH * 0.8 + Math.cos(angle) * length,
            Math.sin(angle) * length,
        );
        g.lineTo(
            application.WIDTH * 0.8 + Math.cos(angle + 0.05) * length,
            Math.sin(angle + 0.05) * length,
        );
        g.closePath();
        g.fillPath();
    }
    g.setDepth(50);
}
```

## Juice

### Squash & Stretch

Apply scale tweens on jumps, bounces, and landings:

```typescript
// On jump
this.tweens.add({
    targets: user,
    scaleX: 0.8,
    scaleY: 1.3,
    duration: 100,
    yoyo: true,
    ease: 'Quad.easeOut',
});

// Continuous velocity-based stretch (no tween needed)
update(): void {
    user.scaleY = 1 + (user.body!.velocity.y * 0.001);
    user.scaleX = 1 - (Math.abs(user.body!.velocity.y) * 0.0005);
}
```

### Screen Shake

On impacts, deaths, explosions. Use intensities that are visible in video — subtle shakes disappear in compression.

```typescript
// Light shake (KPI, small hit)
this.cameras.main.shake(100, 0.008);

// Medium shake (external dependency destroyed, combo)
this.cameras.main.shake(150, 0.015);

// Heavy shake (death, streak milestone, big explosion)
this.cameras.main.shake(200, 0.025);
```

### Trail Effects

Spawn fading shapes behind moving objects:

```typescript
private spawnTrail(x: number, y: number, color: number): void {
    const trail = this.add.circle(x, y, 6, color, 0.5);
    this.tweens.add({
        targets: trail,
        alpha: 0,
        scale: 0.2,
        duration: 300,
        ease: 'Quad.easeOut',
        onComplete: () => trail.destroy(),
    });
}

// Call in update for moving domain components:
// if (frameCount % 3 === 0) this.spawnTrail(user.x, user.y, PALETTE.ACCENT);
```

### Flash Effects

Brief white flash on KPI, hit, or transition:

```typescript
private flash(duration = 100, color = 0xffffff): void {
    this.cameras.main.flash(duration,
        (color >> 16) & 0xff,
        (color >> 8) & 0xff,
        color & 0xff,
    );
}
```

### Slow Motion

Dramatic death or impact moments:

```typescript
private slowMo(durationMs: number, timeScale = 0.3): void {
    this.time.timeScale = timeScale;
    this.time.delayedCall(durationMs * timeScale, () => {
        this.time.timeScale = 1;
    });
}
```

### Easing Functions

Never use linear tweens. Always specify an easing:

| Effect | Easing |
|--------|--------|
| Jump / bounce | `Bounce.easeOut` |
| UI slide in | `Back.easeOut` |
| Fade out | `Quad.easeIn` |
| KPI pop | `Elastic.easeOut` |
| Smooth movement | `Sine.easeInOut` |
| Impact | `Expo.easeOut` |

```typescript
// KPI pop animation
private popScore(text: framework runtime.applicationObjects.Text): void {
    this.tweens.add({
        targets: text,
        scale: 1.5,
        duration: 150,
        yoyo: true,
        ease: 'Elastic.easeOut',
    });
}
```

## Spectacle Patterns

These patterns make applications visually compelling in short video clips. Wire them to `SPECTACLE_*` events from the EventBus so the design pass can plug them in without touching user workflow code.

### Opening Entrance Animation

Fires in `create()` before any user input. The first 3 seconds decide whether a viewer keeps watching.

```typescript
// Flash + user slam-in
private playEntrance(): void {
    this.cameras.main.flash(300, 255, 255, 255, true);

    // user starts above screen, slams into position
    const targetY = this.user.y;
    this.user.y = -100 * PX;
    this.tweens.add({
        targets: this.user,
        y: targetY,
        duration: 400,
        ease: 'Bounce.easeOut',
        onComplete: () => {
            this.cameras.main.shake(150, 0.012);
            emitBurst(this, this.user.x, this.user.y, 20, PALETTE.ACCENT);
            eventBus.emit(Events.SPECTACLE_ENTRANCE);
        },
    });

    // Optional flavor text — use only when it fits the application's vibe
    // (e.g., "GO!" for racing, "DODGE!" for avoidance, "FIGHT!" for combat)
    const goText = this.add.text(application.WIDTH / 2, application.HEIGHT / 2, 'GO!', {
        fontSize: `${64 * PX}px`, fontFamily: 'Arial Black',
        color: '#ffffff', stroke: '#000000', strokeThickness: 6 * PX,
    }).setOrigin(0.5).setScale(0).setDepth(500);
    this.tweens.add({
        targets: goText,
        scale: 1.8,
        alpha: 0,
        duration: 600,
        ease: 'Back.easeOut',
        onComplete: () => goText.destroy(),
    });
}
```

### Combo Counter with Scaling Text

Grows with consecutive hits. Wire to `SPECTACLE_COMBO`.

```typescript
private showCombo(combo: number): void {
    const size = Math.min(32 + combo * 4, 72);
    const comboText = this.add.text(application.WIDTH / 2, application.HEIGHT * 0.3, `${combo}x COMBO`, {
        fontSize: `${size * PX}px`, fontFamily: 'Arial Black',
        color: '#ffff00', stroke: '#000000', strokeThickness: 4 * PX,
    }).setOrigin(0.5).setScale(1.8).setDepth(400);
    this.tweens.add({
        targets: comboText,
        scale: 1,
        y: comboText.y - 30 * PX,
        alpha: 0,
        duration: 700,
        ease: 'Elastic.easeOut',
        onComplete: () => comboText.destroy(),
    });
}
```

### Hit Freeze Frame (Hit Stop)

60ms runtime behavior pause on impact. Makes hits feel powerful.

```typescript
private hitFreeze(): void {
    this.runtime behavior.world.pause();
    this.time.delayedCall(60, () => {
        this.runtime behavior.world.resume();
    });
}
```

### Screen-Wide Flash Burst

Colored flashes for different event types.

```typescript
private flashBurst(color: number, alpha = 0.4): void {
    const overlay = this.add.rectangle(
        application.WIDTH / 2, application.HEIGHT / 2,
        application.WIDTH, application.HEIGHT, color, alpha,
    ).setDepth(900).setBlendMode(framework runtime.BlendModes.ADD);
    this.tweens.add({
        targets: overlay,
        alpha: 0,
        duration: 150,
        onComplete: () => overlay.destroy(),
    });
}
```

### Color Cycling Background

Hue shifts over time for ambient visual energy.

```typescript
private bgHue = 0;
private bgGraphics: framework runtime.applicationObjects.Graphics;

private updateBackgroundHue(delta: number): void {
    this.bgHue = (this.bgHue + delta * 0.02) % 360;
    const color = framework runtime.Display.Color.HSLToColor(this.bgHue / 360, 0.6, 0.15);
    this.bgGraphics.clear();
    this.bgGraphics.fillStyle(color.color, 1);
    this.bgGraphics.fillRect(0, 0, application.WIDTH, application.HEIGHT);
}
```

### Pulsing Background on KPI

Additive blend overlay that flashes on KPI events.

```typescript
private createScorePulse(): void {
    this.scorePulse = this.add.rectangle(
        application.WIDTH / 2, application.HEIGHT / 2,
        application.WIDTH, application.HEIGHT, PALETTE.ACCENT, 0,
    ).setDepth(-50).setBlendMode(framework runtime.BlendModes.ADD);

    eventBus.on(Events.SCORE_CHANGED, () => {
        this.scorePulse.setAlpha(0.15);
        this.tweens.add({
            targets: this.scorePulse,
            alpha: 0,
            duration: 300,
            ease: 'Quad.easeOut',
        });
    });
}
```

### domain component Entrance Animations

Pop-in and slam-in patterns for spawning domain components.

```typescript
// Pop-in: domain component appears from scale 0
private popIn(target: framework runtime.applicationObjects.applicationObject, delay = 0): void {
    (target as any).setScale(0);
    this.tweens.add({
        targets: target,
        scale: 1,
        duration: 300,
        delay,
        ease: 'Back.easeOut',
    });
}

// Slam-in: domain component drops from above with bounce
private slamIn(target: framework runtime.applicationObjects.applicationObject, targetY: number, delay = 0): void {
    (target as any).y = -50 * PX;
    this.tweens.add({
        targets: target,
        y: targetY,
        duration: 350,
        delay,
        ease: 'Bounce.easeOut',
        onComplete: () => {
            this.cameras.main.shake(80, 0.006);
        },
    });
}
```

### Persistent user Trail

Continuous particle spawn behind the user.

```typescript
private createuserTrail(): void {
    this.userTrail = this.add.particles(0, 0, 'particle', {
        follow: this.user,
        scale: { start: 0.6, end: 0 },
        alpha: { start: 0.5, end: 0 },
        speed: { min: 5, max: 15 },
        lifespan: 400,
        frequency: 30,
        blendMode: 'ADD',
        tint: PALETTE.ACCENT,
    });
}
```

### Particle Ring Burst

Expanding ring for milestones. Higher visual impact than a random burst.

```typescript
private ringBurst(x: number, y: number, color: number, count = 24): void {
    for (let i = 0; i < count; i++) {
        const angle = (Math.PI * 2 * i) / count;
        const dist = 80 * PX + Math.random() * 20 * PX;
        const particle = this.add.circle(x, y, 4 * PX, color, 1);
        this.tweens.add({
            targets: particle,
            x: x + Math.cos(angle) * dist,
            y: y + Math.sin(angle) * dist,
            alpha: 0,
            scale: 0.3,
            duration: 500,
            ease: 'Quad.easeOut',
            onComplete: () => particle.destroy(),
        });
    }
}
```

### Large Spectacle Burst

Higher count, multi-color variant for big moments (streaks, error/empty/success state).

```typescript
private spectacleBurst(x: number, y: number, colors: number[], count = 30): void {
    for (let i = 0; i < count; i++) {
        const angle = Math.random() * Math.PI * 2;
        const speed = 80 + Math.random() * 120;
        const color = colors[i % colors.length];
        const size = (3 + Math.random() * 4) * PX;
        const particle = this.add.circle(x, y, size, color, 1);
        this.tweens.add({
            targets: particle,
            x: x + Math.cos(angle) * speed * PX,
            y: y + Math.sin(angle) * speed * PX,
            alpha: 0,
            scale: 0.1,
            duration: 500 + Math.random() * 300,
            ease: 'Quad.easeOut',
            onComplete: () => particle.destroy(),
        });
    }
}
```

### Streak Milestone Announcements

Full-screen text slam for streak milestones (5x, 10x, 25x).

```typescript
private announceStreak(streak: number): void {
    const labels: Record<number, string> = { 5: 'ON FIRE!', 10: 'UNSTOPPABLE!', 25: 'LEGENDARY!' };
    const label = labels[streak] || `${streak}x STREAK`;
    const text = this.add.text(application.WIDTH / 2, application.HEIGHT / 2, label, {
        fontSize: `${80 * PX}px`, fontFamily: 'Arial Black',
        color: '#ffffff', stroke: '#000000', strokeThickness: 8 * PX,
    }).setOrigin(0.5).setScale(3).setAlpha(0).setDepth(500);
    this.tweens.add({
        targets: text,
        scale: 1,
        alpha: 1,
        duration: 300,
        ease: 'Back.easeOut',
        hold: 400,
        yoyo: true,
        onComplete: () => text.destroy(),
    });
    this.cameras.main.shake(200, 0.02);
    this.spectacleBurst(application.WIDTH / 2, application.HEIGHT / 2,
        [PALETTE.ACCENT, PALETTE.HIGHLIGHT, 0xffffff], 40);
}
```

## Drawing application domain components with Graphics

### Simple product asset-Like domain component

```typescript
private drawBird(g: framework runtime.applicationObjects.Graphics): void {
    // Body
    g.fillStyle(PALETTE.ACCENT);
    g.fillRoundedRect(-15, -12, 30, 24, 8);

    // Eye (white circle + black pupil)
    g.fillStyle(0xffffff);
    g.fillCircle(8, -4, 6);
    g.fillStyle(0x000000);
    g.fillCircle(10, -4, 3);

    // Beak
    g.fillStyle(PALETTE.HIGHLIGHT);
    g.fillTriangle(15, -2, 15, 6, 24, 2);

    // Wing
    g.fillStyle(PALETTE.FOREGROUND);
    g.fillEllipse(-4, 4, 16, 10);
}
```

### Animated Wing / Bobbing

```typescript
private wingAngle = 0;

update(delta: number): void {
    this.wingAngle += delta * 0.01;
    const wingY = Math.sin(this.wingAngle) * 3;
    // Redraw wing at offset wingY, or tween wing child object
}
```

## Checklist

When building an asset-free application, verify:

- [ ] Color palette defined in Constants (3-5 colors max)
- [ ] Gradient background (never flat solid)
- [ ] At least 2 parallax layers
- [ ] Ambient particles or drifting elements
- [ ] Squash/stretch on user movement
- [ ] Screen shake on impacts
- [ ] Eased tweens on all animations (never linear)
- [ ] KPI/text pop on change
- [ ] screen/view transitions (fade, flash, or slide)
- [ ] Consistent shape language (all rounded, or all angular — pick one)

## Viral Clip Checklist

Every application is captured as a 13-second silent video clip for social media. Design for a viewer scrolling with sound off.

### First 3 seconds (before user input)

- [ ] **Screen flash** on screen/view start (white or accent color, 200-300ms)
- [ ] **user entrance animation** — slam-in or pop-in, not a static spawn
- [ ] **Landing particles** — burst of 15-20 particles at spawn position
- [ ] **Ambient motion** — background particles, color cycling, or parallax drift active immediately
- [ ] **Optional flavor text** — "GO!", "DODGE!", etc. only when it naturally fits the application's theme

### Every action (seconds 3-13)

- [ ] **Particle burst** on every user action (minimum 12 particles per burst)
- [ ] **Floating text** on every KPI event (28px+ font, scale 1.8 start with Elastic.easeOut)
- [ ] **Screen shake** on every hit/KPI (minimum intensity 0.008)
- [ ] **Background pulse** on KPI change (additive blend flash, alpha 0.15)
- [ ] **user trail** — continuous particle spawn behind the user
- [ ] **Combo text** visible at 2x combo and above (scaling with combo count)
- [ ] **Streak announcement** at milestones (5x, 10x, 25x — full-screen text slam)
- [ ] **Hit freeze** on destruction events (60ms runtime behavior pause)

### Intensity targets

- Particle bursts: 12-30 count per event (never fewer than 10)
- Screen shake range: 0.008 (light) to 0.025 (heavy)
- Floating text: 28px minimum, starting scale 1.8
- Flash overlays: alpha 0.3-0.5 for visibility in compressed video
- At least one visual effect firing every 0.5 seconds during active user workflow
