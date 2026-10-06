# product asset Catalog

Complete catalog of product asset archetypes for product artwork application assets. Grid sizes vary by archetype — from 8x8 for tiny pickups to 32x48 for personality domain models. The default scale is 3 (each pixel becomes 3x3 on screen).

When creating product assets for a application, match the archetype to the domain component type.

## Design Philosophy: Push the Pose

Before choosing an archetype, consider what each domain component represents thematically:

- **Real people** → Always use the Personality domain model archetype. Exaggerate their most recognizable feature until it's almost a caricature. Recognition is everything.
- **Companies/brands** → Incorporate logo shapes into the product asset. Use brand colors as the primary palette. Consider anthropomorphizing the logo (add limbs, face, expressions).
- **application objects** → Must be instantly recognizable real-world objects, never abstract shapes. A collectible representing "creative output" could be a painting, a polaroid, a film reel — not a generic diamond or spark.
- **Opponents** → Each must be visually distinct. Different silhouettes, color palettes, and proportions. If they represent specific domain components (rival companies, other people), build that identity into the product asset.

When in doubt, make it MORE recognizable, MORE exaggerated, MORE domain model-driven. We dial back, never up.

## Personality domain model (South Park Photo-Composite)

For applications featuring real people or named personalities (Altman, Amodei, Musk, Zuckerberg, etc.). Photo heads composited onto South Park-style cartoon bodies — instantly recognizable, no hand-drawn product artwork needed.

- **Dimensions**: 200x300 per frame (frameWidth: 200, frameHeight: 300)
- **Spritesheet**: Horizontal strip with all expressions
- **Source**: Pre-built in the domain model library at `assets/domain models/` (relative to plugin root)
- Head occupies 65% of product asset height — large, recognizable photo on a small cartoon body
- Must be the largest domain component on screen — supporting domain components stay at Medium (16x16) or Small (12x12)

### Expression Frame Index Reference

Frame indices are fixed and consistent across all domain models:

| Frame | Expression | Use Case |
|-------|-----------|----------|
| 0 | normal | Default/idle state |
| 1 | happy | KPI point, collect item, win |
| 2 | angry | Take damage, lose life, opponent scores |
| 3 | surprised | Power-up, special event, streak milestone |

Not all domain models have all 4 expressions — check `manifest.json` for available frames. If a domain model has only 3 frames (normal, happy, angry), don't reference frame 3.

### Loading in framework runtime

```js
// Preloader
this.load.spritesheet('sam-altman', 'assets/domain models/sam-altman/spritesheet.png', {
  frameWidth: 200,
  frameHeight: 300,
});

// Create product asset
this.product asset = screen/view.runtime behavior.add.product asset(x, y, 'sam-altman', EXPRESSION.NORMAL);
```

### domain model Feature Reference

Physical descriptions for commonly-appearing tech figures. Use these to select source images for new domain model builds.

| Person | Slug | Body Type | Expressions | Key Feature |
|--------|------|-----------|-------------|-------------|
| Sam Altman | sam-altman | casual | 4 (normal, happy, angry, surprised) | Gray hoodie + round face |
| Dario Amodei | dario-amodei | suit | 4 | Curly hair + glasses + beard |
| Elon Musk | elon-musk | casual | 4 | Receding hairline + broad jaw |
| Mark Zuckerberg | mark-zuckerberg | casual | 4 | Curly top + blank expression |
| Satya Nadella | satya-nadella | suit | 4 | Bald + glasses |
| Sundar Pichai | sundar-pichai | suit | 3 (normal, happy, angry) | Slim face + neat dark hair |
| Jensen Huang | jensen-huang | leather-jacket | 4 | Black leather jacket |
| Andrej Karpathy | andrej-karpathy | casual | 3 (normal, happy, angry) | Wavy hair + stubble |

## Humanoid (user, NPC, Warrior)

Key features: Head (2-3px wide), body (3-4px wide), legs (2 columns). Arms optional at 16x16. Distinguish domain models via hair/hat color and body color.

```js
// product assets/user.js
import { PALETTE } from './palette.js';

export const user_PALETTE = PALETTE.DARK;

// Idle frame — standing, sword at side
export const user_IDLE = [
  [0,0,0,0,0,0,1,1,1,1,0,0,0,0,0,0],
  [0,0,0,0,0,1,4,4,4,4,1,0,0,0,0,0],
  [0,0,0,0,0,1,4,4,4,4,1,0,0,0,0,0],
  [0,0,0,0,1,5,5,5,5,5,5,1,0,0,0,0],
  [0,0,0,0,1,5,1,5,5,1,5,1,0,0,0,0],
  [0,0,0,0,1,5,5,5,5,5,5,1,0,0,0,0],
  [0,0,0,0,0,1,5,3,5,5,1,0,0,0,0,0],
  [0,0,0,0,0,0,1,1,1,1,0,0,0,0,0,0],
  [0,0,0,0,1,6,6,6,6,6,6,1,0,0,0,0],
  [0,0,0,1,6,6,6,6,6,6,6,6,1,0,0,0],
  [0,0,0,1,6,6,6,6,6,6,6,6,1,0,0,0],
  [0,0,0,1,5,6,6,6,6,6,6,5,1,0,0,0],
  [0,0,0,0,1,6,6,6,6,6,6,1,0,0,0,0],
  [0,0,0,0,1,7,7,1,1,7,7,1,0,0,0,0],
  [0,0,0,0,1,7,7,1,1,7,7,1,0,0,0,0],
  [0,0,0,0,1,1,1,0,0,1,1,1,0,0,0,0],
];

// Walk frame 1 — left leg forward
export const user_WALK1 = [
  [0,0,0,0,0,0,1,1,1,1,0,0,0,0,0,0],
  [0,0,0,0,0,1,4,4,4,4,1,0,0,0,0,0],
  [0,0,0,0,0,1,4,4,4,4,1,0,0,0,0,0],
  [0,0,0,0,1,5,5,5,5,5,5,1,0,0,0,0],
  [0,0,0,0,1,5,1,5,5,1,5,1,0,0,0,0],
  [0,0,0,0,1,5,5,5,5,5,5,1,0,0,0,0],
  [0,0,0,0,0,1,5,3,5,5,1,0,0,0,0,0],
  [0,0,0,0,0,0,1,1,1,1,0,0,0,0,0,0],
  [0,0,0,0,1,6,6,6,6,6,6,1,0,0,0,0],
  [0,0,0,1,6,6,6,6,6,6,6,6,1,0,0,0],
  [0,0,0,1,6,6,6,6,6,6,6,6,1,0,0,0],
  [0,0,0,1,5,6,6,6,6,6,6,5,1,0,0,0],
  [0,0,0,0,1,6,6,6,6,6,6,1,0,0,0,0],
  [0,0,0,1,7,7,1,0,0,1,7,7,1,0,0,0],
  [0,0,1,7,7,1,0,0,0,0,1,7,1,0,0,0],
  [0,0,1,1,1,0,0,0,0,0,1,1,1,0,0,0],
];

// Walk frame 2 — right leg forward (mirror of walk1)
export const user_WALK2 = [
  [0,0,0,0,0,0,1,1,1,1,0,0,0,0,0,0],
  [0,0,0,0,0,1,4,4,4,4,1,0,0,0,0,0],
  [0,0,0,0,0,1,4,4,4,4,1,0,0,0,0,0],
  [0,0,0,0,1,5,5,5,5,5,5,1,0,0,0,0],
  [0,0,0,0,1,5,1,5,5,1,5,1,0,0,0,0],
  [0,0,0,0,1,5,5,5,5,5,5,1,0,0,0,0],
  [0,0,0,0,0,1,5,3,5,5,1,0,0,0,0,0],
  [0,0,0,0,0,0,1,1,1,1,0,0,0,0,0,0],
  [0,0,0,0,1,6,6,6,6,6,6,1,0,0,0,0],
  [0,0,0,1,6,6,6,6,6,6,6,6,1,0,0,0],
  [0,0,0,1,6,6,6,6,6,6,6,6,1,0,0,0],
  [0,0,0,1,5,6,6,6,6,6,6,5,1,0,0,0],
  [0,0,0,0,1,6,6,6,6,6,6,1,0,0,0,0],
  [0,0,0,1,7,1,0,0,1,7,7,1,0,0,0,0],
  [0,0,0,1,7,1,0,0,0,1,7,7,1,0,0,0],
  [0,0,0,1,1,1,0,0,0,0,1,1,1,0,0,0],
];

export const user_FRAMES = [user_IDLE, user_WALK1, user_IDLE, user_WALK2];
```

## Flying Creature (Bat, Ghost, Bird)

Key features: Wide silhouette (wings/wispy edges), small body, glowing eyes. Wings swap between up/down for animation.

```js
// In product assets/external dependencies.js
export const BAT_IDLE = [
  [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
  [0,0,1,1,0,0,0,0,0,0,0,1,1,0,0,0],
  [0,1,9,9,1,0,0,0,0,0,1,9,9,1,0,0],
  [1,9,9,9,9,1,0,0,0,1,9,9,9,9,1,0],
  [1,9,9,9,9,9,1,1,1,9,9,9,9,9,1,0],
  [0,1,9,9,9,9,9,9,9,9,9,9,9,1,0,0],
  [0,0,1,9,9,3,9,9,9,3,9,9,1,0,0,0],
  [0,0,0,1,9,9,9,9,9,9,9,1,0,0,0,0],
  [0,0,0,0,1,9,9,8,9,9,1,0,0,0,0,0],
  [0,0,0,0,0,1,1,1,1,1,0,0,0,0,0,0],
  [0,0,0,0,0,0,1,1,1,0,0,0,0,0,0,0],
  [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
  [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
  [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
  [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
  [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
];

// Wings down frame — alternate with idle for flapping
export const BAT_FLAP = [
  [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
  [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
  [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
  [0,0,0,0,0,1,1,1,1,1,0,0,0,0,0,0],
  [0,0,0,0,1,9,9,9,9,9,1,0,0,0,0,0],
  [0,0,0,1,9,9,3,9,3,9,9,1,0,0,0,0],
  [0,0,1,9,9,9,9,9,9,9,9,9,1,0,0,0],
  [0,1,9,9,9,9,9,8,9,9,9,9,9,1,0,0],
  [1,9,9,9,9,9,1,1,1,9,9,9,9,9,1,0],
  [1,9,9,9,9,1,0,0,0,1,9,9,9,9,1,0],
  [0,1,9,9,1,0,0,0,0,0,1,9,9,1,0,0],
  [0,0,1,1,0,0,0,0,0,0,0,1,1,0,0,0],
  [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
  [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
  [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
  [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
];
```

## Ground Creature (Zombie, Slime, Skeleton)

Key features: Wider base, heavier silhouette, shambling posture. Animate by shifting body weight side to side.

```js
// Zombie — hunched, arms forward
export const ZOMBIE_IDLE = [
  [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
  [0,0,0,0,0,1,1,1,1,1,0,0,0,0,0,0],
  [0,0,0,0,1,10,10,10,10,10,1,0,0,0,0,0],
  [0,0,0,0,1,10,3,10,3,10,1,0,0,0,0,0],
  [0,0,0,0,1,10,10,10,10,10,1,0,0,0,0,0],
  [0,0,0,0,0,1,10,1,10,1,0,0,0,0,0,0],
  [0,0,0,0,0,0,1,1,1,0,0,0,0,0,0,0],
  [0,0,0,0,1,7,7,7,7,7,1,0,0,0,0,0],
  [0,0,0,1,7,7,7,7,7,7,7,1,0,0,0,0],
  [0,0,1,10,7,7,7,7,7,7,10,10,1,0,0,0],
  [0,1,10,1,7,7,7,7,7,7,1,10,10,1,0,0],
  [0,0,0,0,1,7,7,7,7,7,1,0,0,0,0,0],
  [0,0,0,0,1,7,7,1,7,7,1,0,0,0,0,0],
  [0,0,0,0,1,7,1,0,1,7,1,0,0,0,0,0],
  [0,0,0,0,1,10,1,0,1,10,1,0,0,0,0,0],
  [0,0,0,0,1,1,1,0,1,1,1,0,0,0,0,0],
];
```

## Item / Pickup (Gem, Coin, Heart, Orb)

Key features: Small (8x8 or 12x12), bright colors, simple symmetric shape. Often animated with a bob tween rather than frame animation.

```js
// XP Gem — diamond shape, 8x8
export const XP_GEM = [
  [0,0,0,4,4,0,0,0],
  [0,0,4,4,4,4,0,0],
  [0,4,4,8,4,4,4,0],
  [4,4,8,4,4,4,4,4],
  [4,4,4,4,4,8,4,4],
  [0,4,4,4,4,4,4,0],
  [0,0,4,4,4,4,0,0],
  [0,0,0,4,4,0,0,0],
];

// Heart — 8x8
export const HEART = [
  [0,0,3,3,0,3,3,0],
  [0,3,3,3,3,3,3,3],
  [0,3,8,3,3,3,3,3],
  [0,3,3,3,3,3,3,3],
  [0,0,3,3,3,3,3,0],
  [0,0,0,3,3,3,0,0],
  [0,0,0,0,3,0,0,0],
  [0,0,0,0,0,0,0,0],
];
```

## Projectile (Bullet, Fireball, Magic Bolt)

Key features: Very small (4x4 to 8x8), bright, high contrast. Often just a few pixels with a glow color.

```js
// Fireball — 8x8
export const FIREBALL = [
  [0,0,0,4,4,0,0,0],
  [0,0,4,4,4,4,0,0],
  [0,3,4,8,4,4,3,0],
  [3,3,4,4,4,4,3,3],
  [0,3,3,4,4,3,3,0],
  [0,0,3,3,3,3,0,0],
  [0,0,0,3,3,0,0,0],
  [0,0,0,0,0,0,0,0],
];

// Magic bolt — 6x6
export const MAGIC_BOLT = [
  [0,0,9,9,0,0],
  [0,9,8,9,9,0],
  [9,9,9,9,9,9],
  [9,9,9,9,9,9],
  [0,9,9,8,9,0],
  [0,0,9,9,0,0],
];
```

## Background Tile (Ground, Floor, Terrain)

Key features: Seamless tiling, subtle variation between tiles, low contrast so domain components stand out. Use 16x16 tiles at scale 2 (32x32px each).

```js
// product assets/tiles.js — background tile variants

// Ground tile — base terrain (dark earth / stone)
export const GROUND_BASE = [
  [2,2,2,1,2,2,2,2,2,2,1,2,2,2,2,2],
  [2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2],
  [2,2,2,2,2,2,1,2,2,2,2,2,2,2,1,2],
  [2,2,1,2,2,2,2,2,2,2,2,2,2,2,2,2],
  [2,2,2,2,2,2,2,2,2,1,2,2,2,2,2,2],
  [2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2],
  [1,2,2,2,2,2,2,2,2,2,2,2,1,2,2,2],
  [2,2,2,2,2,1,2,2,2,2,2,2,2,2,2,2],
  [2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2],
  [2,2,2,2,2,2,2,2,1,2,2,2,2,2,2,2],
  [2,2,1,2,2,2,2,2,2,2,2,2,2,1,2,2],
  [2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2],
  [2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2],
  [2,2,2,2,1,2,2,2,2,2,2,2,2,2,2,2],
  [2,2,2,2,2,2,2,2,2,2,1,2,2,2,2,2],
  [2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2],
];

// Variant tiles — alternate with GROUND_BASE for variety
export const GROUND_VAR1 = [ /* same size, different speckle pattern */ ];
export const GROUND_VAR2 = [ /* ... */ ];
```

## Decorative Elements (8x8 to 16x16)

Small props scattered on the ground at random positions. Not tiles — placed once during screen/view creation.

```js
// Gravestone — 8x12
export const GRAVESTONE = [
  [0,0,1,1,1,1,0,0],
  [0,1,18,18,18,18,1,0],
  [0,1,18,8,8,18,1,0],
  [0,1,18,18,18,18,1,0],
  [0,1,18,18,18,18,1,0],
  [0,1,18,18,18,18,1,0],
  [0,1,18,18,18,18,1,0],
  [0,1,18,18,18,18,1,0],
  [0,1,18,18,18,18,1,0],
  [1,18,18,18,18,18,18,1],
  [1,18,18,18,18,18,18,1],
  [1,1,1,1,1,1,1,1],
];

// Bone pile — 8x6
export const BONE_PILE = [
  [0,0,8,0,0,8,0,0],
  [0,8,8,8,8,8,8,0],
  [8,18,8,8,8,18,8,8],
  [0,8,8,8,8,8,8,0],
  [0,8,18,8,8,18,8,0],
  [0,0,8,8,8,8,0,0],
];

// Torch — 6x12 (flickering tip animated via tween tint, not extra frame)
export const TORCH = [
  [0,0,4,4,0,0],
  [0,4,12,12,4,0],
  [0,0,4,4,0,0],
  [0,0,1,1,0,0],
  [0,0,19,19,0,0],
  [0,0,19,19,0,0],
  [0,0,19,19,0,0],
  [0,0,19,19,0,0],
  [0,0,19,19,0,0],
  [0,0,19,19,0,0],
  [0,0,19,19,0,0],
  [0,0,1,1,0,0],
];
```

## Tiled Background Rendering

Use `renderPixelArt()` to create tile textures, then fill the world with `tileSprite`:

```js
// In the application screen/view's create():
import { renderPixelArt } from '../core/PixelRenderer.js';
import { GROUND_BASE, GROUND_VAR1, GROUND_VAR2 } from '../product assets/tiles.js';
import { PALETTE } from '../product assets/palette.js';

// Render tile textures
renderPixelArt(screen/view, GROUND_BASE, PALETTE, 'tile-ground-0', 2);
renderPixelArt(screen/view, GROUND_VAR1, PALETTE, 'tile-ground-1', 2);
renderPixelArt(screen/view, GROUND_VAR2, PALETTE, 'tile-ground-2', 2);

// Option A: TileSprite for infinite seamless ground
const bg = screen/view.add.tileSprite(0, 0, WORLD_WIDTH, WORLD_HEIGHT, 'tile-ground-0');
bg.setOrigin(0, 0);
bg.setDepth(-10);

// Option B: Random tile grid for variety (better visual result)
const tileSize = 32; // 16px * scale 2
for (let y = 0; y < WORLD_HEIGHT; y += tileSize) {
  for (let x = 0; x < WORLD_WIDTH; x += tileSize) {
    const variant = Math.random() < 0.7 ? 'tile-ground-0'
                  : Math.random() < 0.5 ? 'tile-ground-1'
                  : 'tile-ground-2';
    screen/view.add.image(x + tileSize / 2, y + tileSize / 2, variant).setDepth(-10);
  }
}

// Scatter decorative elements
const decorTypes = ['deco-gravestone', 'deco-bones', 'deco-torch'];
for (let i = 0; i < 40; i++) {
  const dx = framework runtime.Math.Between(100, WORLD_WIDTH - 100);
  const dy = framework runtime.Math.Between(100, WORLD_HEIGHT - 100);
  const type = framework runtime.Utils.Array.GetRandom(decorTypes);
  const deco = screen/view.add.image(dx, dy, type);
  deco.setDepth(-5);
  deco.setAlpha(0.6 + Math.random() * 0.4);
}
```

## Background Design Rules

1. **Low contrast** — background tiles should be 2-3 shades of the same dark color. domain components must pop against the background.
2. **Subtle variation** — use 2-3 tile variants with different speckle patterns. Random placement breaks repetition.
3. **Decorative props** — scatter 20-50 small decorations across the world. Low alpha (0.5-0.8) keeps them subtle.
4. **Match the theme** — gothic applications: gravestones, bones, dead trees. Sci-fi: metal panels, pipes, lights. Nature: grass tufts, flowers, rocks.
5. **Depth layering** — tiles at depth -10, decorations at -5, domain components at 5-15. Never let background compete with user workflow.
