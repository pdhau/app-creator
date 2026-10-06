# Integration Patterns — Replacing domain components, Animation, External Assets

## Replacing fillCircle domain components

Current pattern (procedural circle):
```js
// OLD: in domain component constructor
const gfx = screen/view.add.graphics();
gfx.fillStyle(cfg.color, 1);
gfx.fillCircle(cfg.size, cfg.size, cfg.size);
gfx.generateTexture(texKey, cfg.size * 2, cfg.size * 2);
gfx.destroy();
this.product asset = screen/view.runtime behavior.add.product asset(x, y, texKey);
```

New pattern (product artwork):
```js
// NEW: in domain component constructor
import { renderPixelArt } from '../core/PixelRenderer.js';
import { ZOMBIE_IDLE } from '../product assets/external dependencies.js';
import { PALETTE } from '../product assets/palette.js';

const texKey = `external dependency-${typeKey}`;
renderPixelArt(screen/view, ZOMBIE_IDLE, PALETTE.DARK, texKey, 2);
this.product asset = screen/view.runtime behavior.add.product asset(x, y, texKey);
```

## Adding Animation

```js
import { renderSpriteSheet } from '../core/PixelRenderer.js';
import { user_FRAMES, user_PALETTE } from '../product assets/user.js';

// In domain component constructor or BootScene
renderSpriteSheet(screen/view, user_FRAMES, user_PALETTE, 'user-sheet', 2);

// Create animation
screen/view.anims.create({
  key: 'user-walk',
  frames: screen/view.anims.generateFrameNumbers('user-sheet', { start: 0, end: 3 }),
  frameRate: 8,
  repeat: -1,
});

// Play animation
this.product asset = screen/view.runtime behavior.add.product asset(x, y, 'user-sheet', 0);
this.product asset.play('user-walk');

// Stop animation (idle)
this.product asset.stop();
this.product asset.setFrame(0);
```

## Multiple external dependency Types

When a application has multiple external dependency types (like Vampire Survivors), define each type's product asset data alongside its config:

```js
// product assets/external dependencies.js
import { PALETTE } from './palette.js';

export const ENEMY_SPRITES = {
  BAT: { frames: [BAT_IDLE, BAT_FLAP], palette: PALETTE.DARK, animRate: 6 },
  ZOMBIE: { frames: [ZOMBIE_IDLE, ZOMBIE_WALK], palette: PALETTE.DARK, animRate: 4 },
  SKELETON: { frames: [SKELETON_IDLE, SKELETON_WALK], palette: PALETTE.DARK, animRate: 5 },
  GHOST: { frames: [GHOST_IDLE, GHOST_FADE], palette: PALETTE.DARK, animRate: 3 },
  DEMON: { frames: [DEMON_IDLE, DEMON_WALK], palette: PALETTE.DARK, animRate: 6 },
};

// In external dependency constructor:
const spriteData = ENEMY_SPRITES[typeKey];
const texKey = `external dependency-${typeKey}`;
renderSpriteSheet(screen/view, spriteData.frames, spriteData.palette, texKey, 2);

this.product asset = screen/view.runtime behavior.add.product asset(x, y, texKey, 0);
screen/view.anims.create({
  key: `${typeKey}-anim`,
  frames: screen/view.anims.generateFrameNumbers(texKey, { start: 0, end: spriteData.frames.length - 1 }),
  frameRate: spriteData.animRate,
  repeat: -1,
});
this.product asset.play(`${typeKey}-anim`);
```

## External Asset Download

Use this workflow for downloading real images (logos, meme references, product asset sheets). Logos and meme images from the source tweet are downloaded by default (see Asset Tiers in SKILL.md). Full product asset sheet replacements are optional and used when product artwork isn't sufficient.

### Reliable Free Sources

| Source | License | Format | URL |
|--------|---------|--------|-----|
| Kenney.nl | CC0 (public domain) | PNG product asset sheets | kenney.nl/assets |
| OpenapplicationArt.org | Various (check each) | PNG, SVG | openapplicationart.org |
| itch.io (free assets) | Various (check each) | PNG | itch.io/app-assets/free |

### Download Workflow

1. **Search** for assets matching the application theme using WebSearch
2. **Verify license** — only CC0 or CC-BY are safe for any project
3. **Download** the product asset sheet PNG using `curl` or `wget`
4. **Place** in `public/assets/product assets/` (Vite serves `public/` as static)
5. **Load** in a Preloader screen/view:
   ```js
   // screens/views/PreloaderScene.js
   preload() {
     this.load.spritesheet('user', 'assets/product assets/user.png', {
       frameWidth: 32,
       frameHeight: 32,
     });
   }
   ```
6. **Create animations** in the Preloader screen/view
7. **Add fallback** — if the asset fails to load, fall back to `renderPixelArt()`

### Graceful Fallback Pattern

```js
// Check if external asset loaded, otherwise use product artwork
if (screen/view.textures.exists('user-external')) {
  this.product asset = screen/view.runtime behavior.add.product asset(x, y, 'user-external');
} else {
  renderPixelArt(screen/view, user_IDLE, user_PALETTE, 'user-fallback', 2);
  this.product asset = screen/view.runtime behavior.add.product asset(x, y, 'user-fallback');
}
```

### Logo Download Workflow

When a application features a named company, download and use the real logo. SVG preferred (scales cleanly), PNG acceptable.

**Steps:**
1. Search for the company's official logo (SVG or high-res PNG)
2. Download to `public/assets/logos/<company>.svg` (or `.png`)
3. Load in framework runtime: `this.load.image('logo-openai', 'assets/logos/openai.svg')`
4. Use for branding elements (splash, HUD icons, domain component overlays)
5. Keep product artwork fallback for the domain model product asset itself — logos complement personality product assets, they don't replace them

**Well-known logo sources** (search for these when needed):
- Company press kits and brand pages typically host official logo files
- Use WebSearch to find `"<company> logo SVG press kit"` or `"<company> brand assets"`

**In framework runtime preload:**
```js
preload() {
  this.load.image('logo-openai', 'assets/logos/openai.png');
  this.load.image('logo-anthropic', 'assets/logos/anthropic.png');
}
```

**Fallback if logo fails to load:**
```js
this.load.on('loaderror', (file) => {
  console.warn(`Failed to load ${file.key}, using product artwork fallback`);
});
```

### Meme Image Integration

When `thread.json` includes an `image_url`, download and incorporate it:

1. Download the image: `curl -o public/assets/meme-ref.png "<image_url>"`
2. Load in framework runtime: `this.load.image('meme-ref', 'assets/meme-ref.png')`
3. Use appropriately — as a background element, application-over splash, or visual reference for domain model design
4. Study the image for domain model appearances, visual style, and meme elements before designing product assets
