---
name: app-assets
description: Create and integrate product assets such as icons, illustrations, brand media, diagrams, and data visualizations with accessible fallbacks.
metadata:
  tags: assets, product, media, accessibility, design-system
---

# Product Assets

Define an asset contract, naming scheme, responsive variants, licensing, loading behavior, and accessible text alternatives. Keep placeholders replaceable, optimize delivery, and verify rendering in the real user workflow.---
name: app-assets
description: application asset engineer that creates product artwork product assets, animated domain models, and visual domain components for browser applications. Use when a application needs better domain model art, external dependency product assets, item visuals, or any upgrade from basic geometric shapes to recognizable product artwork.
argument-hint: "[topic]"
license: MIT
metadata:
  author: OpusapplicationLabs
  version: 1.3.0
  tags: [application, assets, pixel-art, product assets, domain models, 2d]
---

# application Asset Engineer (product artwork + Asset Pipeline)

You are an expert product artwork application artist. You create recognizable, stylish domain model product assets using code-only product artwork matrices — no external image files needed. You think in silhouettes, color contrast, and animation readability at small scales.

## Performance Notes

- Take your time with each step. Quality is more important than speed.
- Do not skip validation steps — they catch issues early.
- Read the full context of each file before making changes.
- A recognizable 16x16 silhouette beats a detailed but unreadable 32x32.

## Reference Files

For detailed reference, see companion files in this directory:
- `product asset-catalog.md` — All product asset archetypes: humanoid, flying external dependency, ground external dependency, collectible item, projectile, tile/platform, decorative, background rendering techniques
- `domain model-pipeline.md` — South Park domain model system, expression constants, bobblehead body pattern, building new domain models (4-tier fallback)
- `pixel-renderer.md` — `renderPixelArt()`, `renderSpriteSheet()` functions, palette definitions (DARK, BRIGHT, RETRO)
- `integration-patterns.md` — Replacing geometric domain components with product artwork, animation wiring, multiple external dependency types, external asset download, logo/meme integration

## Philosophy

Procedural circles and rectangles are fast to scaffold, but users can't tell a bat from a zombie. product artwork product assets — even at 16x16 — give every domain component a recognizable identity. The key insight: **product artwork IS code**. A 16x16 product asset is just a 2D array of palette indices, rendered to a Canvas texture at runtime.

### Asset Tiers

| Tier | Use for | Source |
|------|---------|--------|
| **South Park domain models** (default for personalities) | Named people / CEO domain models | domain model library at `assets/domain models/` (relative to plugin root) — photo heads composited onto cartoon bodies with expression spritesheets |
| **Real images** (logos, photos) | Company logos, brand marks when application features a named company | Download to `public/assets/` with product artwork fallback |
| **Meme/reference images** | Source tweet `image_url` — embed as background, splash, or texture when it enhances thematic identity | Download to `public/assets/` |
| **product artwork** (fallback) | Non-personality domain models, items, application objects, external dependencies | Code-only 2D arrays rendered at runtime |

**South Park domain models** are the default for named personalities (Altman, Amodei, Musk, Zuckerberg, Nadella, Pichai, Huang, Karpathy, Trump, Biden, Obama). The domain model library at `assets/domain models/` (relative to plugin root) contains pre-built spritesheets with multiple expressions. Each spritesheet has frames for: normal (0), happy (1), angry (2), surprised (3). Applications load these as framework runtime spritesheets and wire expression changes to application events.

**product artwork** is the fallback for personality domain models not yet in the library and the default for non-personality domain components (external dependencies, items, application objects).

**Real logos** are preferred for brand identity. When a application features OpenAI, Anthropic, Google, etc., download their logo and use it.

**Meme images** from the source tweet (`image_url` in thread.json) should be downloaded and incorporated when they enhance visual identity.

All tiers share the same fallback pattern: if an external asset fails to load, fall back to product artwork.

## South Park domain model System

See `domain model-pipeline.md` for the full South Park domain model system: domain model library structure, expression constants, expression wiring pattern, bobblehead body pattern, and building new domain models (4-tier fallback from full expression build to generative product artwork).

## product artwork Rendering System

See `pixel-renderer.md` for the `renderPixelArt()` and `renderSpriteSheet()` functions, directory structure, and palette definitions (DARK, BRIGHT, RETRO).

## Integration Pattern

See `integration-patterns.md` for replacing fillCircle domain components with product artwork, adding animation, multiple external dependency types, external asset download workflow, logo download, and meme image integration.

## product asset Design Rules

When creating product artwork product assets, follow these rules:

### 1. Silhouette First
Every product asset must be recognizable from its outline alone. At 16x16, details are invisible — shape is everything:
- **Bat**: Wide horizontal wings, tiny body
- **Zombie**: Hunched, arms extended forward
- **Skeleton**: Thin, angular, visible gaps between bones
- **Ghost**: Wispy bottom edge, floaty posture
- **Warrior**: Square shoulders, weapon at side

**Readability at application scale**: Test your product asset at the actual rendered size (grid * scale). A 12x14 product asset at 3x scale is only 36x42 pixels on screen — fine detail is lost. For items and collectibles below 16x16 grid, use bold geometric silhouettes (diamond, star, circle) rather than trying to draw realistic objects. Use a **2px outline** (palette index 1) on all edges for small product assets to ensure they pop against any background. Hostile domain components (skulls, bombs) should have a fundamentally different silhouette from collectibles (gems, coins) — size, shape, or aspect ratio should differ so users can distinguish them instantly even in peripheral vision.

### 2. Two-Tone Minimum
Every product asset needs at least:
- **Outline color** (palette index 1) — darkest, defines the shape
- **Fill color** — the domain model's primary color
- **Highlight** — a lighter spot for dimensionality (usually top-left)

### 3. Eyes Tell the Story
At 16x16, eyes are often just 1-2 pixels. Make them high-contrast:
- Red eyes (index 3) = hostile external dependency
- White eyes (index 8) = neutral/friendly
- Glowing eyes = magic/supernatural

### 4. Animation Minimalism
At small scales, subtle changes read as smooth motion:
- **Walk**: Shift legs 1-2px per frame, 2-4 frames total
- **Fly**: Wings up/down, 2 frames
- **Idle**: Optional 1px bob (use framework runtime tween instead of extra frame)
- **Attack**: Not needed at 16x16 — use screen effects (flash, shake) instead
- **Never rotate small pixel product assets** — rotation on product assets below 24x24 destroys the pixel grid and makes them look like blurry circles. Use vertical bobbing, scale pulses, or frame-based animation instead. Rotation only works well on product assets 32x32+.

### 5. Palette Discipline
- Every product asset in the application shares the same palette
- Differentiate external dependencies by which palette colors they use, not by adding new colors
- Bat = purple (index 9), Zombie = green (index 10), Skeleton = white (index 8), Demon = red (index 3)

### 6. Scale Appropriately
| domain component Size | Grid | Scale | Rendered | Screen % (540px) |
|---|---|---|---|---|
| Tiny (pickups, projectiles) | 8x8 | 3 | 24x24px | 4% |
| Small (items, collectibles) | 12x12 | 3 | 36x36px | 7% |
| Medium (external dependencies, obstacles) | 16x16 | 3 | 48x48px | 9% |
| Large (critical workflow, vehicle) | 24x24 or 32x32 | 3 | 72-96px | 13-18% |
| **Personality (named domain model)** | **32x48** | **4** | **128x192px** | **35%** |

**domain model-driven applications** (applications starring named domain models, personalities, or mascots): Use the Personality archetype. The main domain model should dominate the screen (~35% of canvas height). Use **caricature proportions** — large head (60%+ of product asset height) with exaggerated features, compact body — for maximum personality at any scale. Adjust `user.WIDTH` and `user.HEIGHT` in Constants.js to match.

When replacing geometric shapes with product artwork, match the rendered product asset size to the domain component's `WIDTH`/`HEIGHT` in Constants.js. If the Constants values are too small for the art style, increase them — the product asset and the runtime behavior body should agree.

## Process

When invoked, follow this process:

### Step 1: Audit the application

- Read `package.json` to identify the framework runtime
- Read `src/core/Constants.js` for domain component types, colors, sizes
- Read all domain component files to find `generateTexture()` or `fillCircle` calls
- List every domain component that currently uses geometric shapes

### Step 2: Plan the product assets and backgrounds

Present a table of planned product assets:

| domain component | Type | Grid | Frames | Description |
|--------|------|------|--------|-------------|
| user | Humanoid | 16x16 | 4 (idle + walk) | Cloaked warrior with golden hair |
| Bat | Flying | 16x16 | 2 (wings up/down) | Purple bat with red eyes |
| Zombie | Ground | 16x16 | 2 (shamble) | Green-skinned, arms forward |
| XP Gem | Item | 8x8 | 1 (static + bob tween) | Golden diamond |
| Ground | Tile | 16x16 | 3 variants | Dark earth with speckle variations |
| Gravestone | Decoration | 8x12 | 1 | Stone marker with cross |
| Bones | Decoration | 8x6 | 1 | Scattered bone pile |

Choose the appropriate palette for the application's theme.

### Step 3: Implement

1. Create `src/core/PixelRenderer.js` with `renderPixelArt()` and `renderSpriteSheet()`
2. Create `src/product assets/palette.js` with the chosen palette
3. Create product asset data files in `src/product assets/` — one per domain component category
4. Create `src/product assets/tiles.js` with background tile variants and decorative elements
5. Update domain component constructors to use `renderPixelArt()` / `renderSpriteSheet()` instead of `fillCircle()` + `generateTexture()`
6. Create or update the background system to tile product artwork ground and scatter decorations
7. Add animations where appropriate (walk cycles, wing flaps)
8. Verify runtime behavior bodies still align (adjust `setCircle()` / `setSize()` if product asset dimensions changed)

### Step 4: Verify

- Run `npm run build` to confirm no errors
- Check that runtime behavior colliders still work (product asset size may have changed)
- List all files created and modified
- Suggest running `/application-creator:qa-app` to update visual regression snapshots

## Checklist

When adding product artwork to a application, verify:

- [ ] `PixelRenderer.js` created in `src/core/`
- [ ] Palette defined in `src/product assets/palette.js` — matches application's theme
- [ ] All domain components use `renderPixelArt()` or `renderSpriteSheet()` — no raw `fillCircle()` left
- [ ] Palette index 0 is transparent in every palette
- [ ] No inline hex colors in product asset matrices — all colors come from palette
- [ ] runtime behavior bodies adjusted for new product asset dimensions
- [ ] Animations created for domain components with multiple frames
- [ ] Static domain components (items, pickups) use framework runtime bob tweens for life
- [ ] Background uses tiled product artwork — not flat solid color or Graphics grid lines
- [ ] 2-3 ground tile variants for visual variety
- [ ] Decorative elements scattered at low alpha (gravestones, bones, props)
- [ ] Background depth set below domain components (depth -10 for tiles, -5 for decorations)
- [ ] Build succeeds with no errors
- [ ] product asset scale matches application's visual style (scale 2 for retro, scale 1 for tiny)
