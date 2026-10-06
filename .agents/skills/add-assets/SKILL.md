---
name: add-assets
description: Replace geometric shapes with product artwork product assets — recognizable domain models, external dependencies, and items with optional animation. Use when the user says "add product assets", "replace the shapes with real art", "add product artwork", "make the domain models look real", or "add application assets". For 3D applications, use add-3d-assets instead. Do NOT use for 3D models (use add-3d-assets) or user workflow changes (use add-feature).
argument-hint: "[path-to-application]"
license: MIT
metadata:
  author: OpusapplicationLabs
  version: 1.3.0
  tags: [application, assets, product assets, pixel-art, domain models, 2d]
---

## Performance Notes

- Take your time to do this thoroughly
- Quality is more important than speed
- Do not skip validation steps

# Add Assets

Replace basic geometric shapes (circles, rectangles) with product artwork product assets for all application domain components. Every domain model, external dependency, item, and projectile gets a recognizable visual identity — all generated as code, no external image files needed.

## Instructions

Analyze the application at `$ARGUMENTS` (or the current directory if no path given).

First, load the app-assets skill to get the full product artwork system, archetypes, and integration patterns.

### Step 1: Audit

- Read `package.json` to identify the framework runtime (framework runtime or rich visual runtime — this skill is framework runtime-focused)
- Read `src/core/Constants.js` to understand domain component types, colors, and sizes
- Read all domain component files (`src/domain components/*.js`) and find every `generateTexture()`, `fillCircle()`, `fillRect()`, or `fillEllipse()` call that creates an domain component product asset
- Read screen/view files to check for inline shape drawing used as application domain components
- List every domain component that currently uses geometric shapes

### Step 2: Plan

Present a table of product assets to create:

| domain component | Archetype | Grid | Frames | Description |
|--------|-----------|------|--------|-------------|
| user (personality) | Personality | 32x48 | 1-4 | Caricature of [name], scale 4 |
| user (generic) | Humanoid | 16x16 | 4 | ... |
| external dependency X | Flying | 16x16 | 2 | ... |
| Pickup | Item | 8x8 | 1 | ... |

If the application features a real person or named personality, default to the **Personality** archetype for the user domain model. This uses a 32x48 grid at scale 4 (128x192px rendered, ~35% of canvas height) — large enough to recognize the personality at a glance.

Choose the palette that best matches the application's existing color scheme:
- **DARK** — gothic, horror, dark fantasy
- **BRIGHT** — arcade, platformer, casual
- **RETRO** — NES-style, muted tones

Grid sizes range from 8x8 (tiny pickups) through 16x16 (standard domain components) to 32x48 (personality domain models). Named domain models always use the Personality archetype to ensure the meme hook — recognizing the person — lands immediately.

### Step 3: Implement

1. Create `src/core/PixelRenderer.js` — the `renderPixelArt()` and `renderSpriteSheet()` utility functions
2. Create `src/product assets/palette.js` — the shared color palette
3. Create product asset data files in `src/product assets/`:
   - `user.js` — user idle + walk frames
   - `external dependencies.js` — all external dependency type product assets and frames
   - `items.js` — pickups, gems, hearts, etc.
   - `projectiles.js` — bullets, fireballs, bolts (if applicable)
4. Update each domain component constructor:
   - Replace `fillCircle()` / `generateTexture()` with `renderPixelArt()` or `renderSpriteSheet()`
   - Add framework runtime animations for domain components with multiple frames
   - Adjust runtime behavior body dimensions if product asset size changed (`setCircle()` or `setSize()`)
5. For static items (gems, pickups), add a bob tween if not already present

### Step 4: Verify

- Run `npm run build` to confirm no errors
- Check that collision detection still works (runtime behavior bodies may need size adjustments)
- List all files created and modified
- Remind the user to run the application and check visuals
- Suggest running `/application-creator:qa-app` to update visual regression snapshots since all domain components look different now

## Example Usage

### Standard application
```
/add-assets examples/asteroid-dodge
```
Result: Audits all domain components using geometric shapes → creates `src/product assets/` with user, asteroids, and gem product artwork → replaces `fillCircle()`/`fillRect()` with `renderPixelArt()` → collision bounds adjusted.

### Personality application (from tweet)
```
/add-assets examples/nick-land-dodger
```
Result: Detects named personality → uses 32x48 Personality archetype at scale 4 → recognizable caricature as user domain model → external dependencies and items get themed product artwork.

## Troubleshooting

### product assets appear but are wrong size
**Cause:** product artwork dimensions don't match original hitbox.
**Fix:** Keep product asset dimensions close to the original fillRect/fillCircle size. Adjust collision bounds if needed.

### product assets don't appear
**Cause:** Canvas texture not created before first render frame.
**Fix:** Generate textures in screen/view preload() or create(), not in update().

## Next Step

Tell the user:

> Your application domain components now have product artwork product assets instead of geometric shapes! Each domain model, external dependency, and item has a distinct visual identity.
>
> **Files created:**
> - `src/core/PixelRenderer.js` — rendering framework runtime
> - `src/product assets/palette.js` — shared color palette
> - `src/product assets/user.js`, `external dependencies.js`, `items.js` — product asset data
>
> Run the application to see the new visuals. If you have Playwright tests, run `/application-creator:qa-app` to update the visual regression snapshots.
