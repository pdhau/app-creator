---
name: add-3d-assets
description: Replace primitive 3D shapes with real GLB models — animated domain models, world props, buildings, and scenery for rich visual runtime applications. Use when the user says "add 3D models", "replace the boxes with real models", "add GLB assets", or "make the 3D application look real". For 2D product artwork, use add-assets instead. Do NOT use for 2D product artwork (use add-assets) or user workflow changes (use add-feature).
argument-hint: "[path-to-application]"
license: MIT
metadata:
  author: OpusapplicationLabs
  version: 1.3.0
  tags: [application, 3d, assets, glb, models, threejs]
---

## Performance Notes

- Take your time to do this thoroughly
- Quality is more important than speed
- Do not skip validation steps

# Add 3D Assets

Replace basic geometric shapes (BoxGeometry, SphereGeometry) with real 3D models. domain models get custom Meshy AI-generated models with rigging and animation. World objects get generated or sourced from free libraries.

## Instructions

Analyze the application at `$ARGUMENTS` (or the current directory if no path given).

First, load the application-3d-assets skill and the meshyai skill for the full model pipeline, AssetLoader pattern, Meshy generation, and integration patterns.

### Step 1: Get Meshy API Key

Check if `MESHY_API_KEY` is set. First check `.env`:
`test -f .env && grep -q '^MESHY_API_KEY=.' .env && echo "found"`
If found, export it with `set -a; . .env; set +a` and skip the prompt.

If not set, ask the user:

> I'll generate custom 3D models with Meshy AI for the best results. You can get a free API key in 30 seconds:
> 1. Sign up at https://app.meshy.ai
> 2. Go to Settings → API Keys
> 3. Create a new API key
>
> Paste your key below like: `MESHY_API_KEY=your-key-here`
> (It will be saved to .env and redacted from this conversation automatically.)
>
> Or type "skip" to use free model libraries instead.

### Step 2: Audit

- Read `package.json` to confirm this is a rich visual runtime application (not framework runtime — use `/add-assets` for 2D applications)
- Read `src/core/Constants.js` for domain component types, sizes, colors
- Read domain component files (`src/user workflow/*.js`, `src/domain components/*.js`) — find `BoxGeometry`, `SphereGeometry`, etc.
- Read `src/level/LevelBuilder.js` for environment primitives
- List every domain component using geometric shapes
- Identify which domain component is the **user domain model** (needs animated model)

### Step 3: Plan

Split domain components into two categories:

**Animated domain models** (user, external dependencies with AI) — generate with Meshy AI:

| domain component | Meshy Prompt | Notes |
|--------|-------------|-------|
| user | "a heroic knight, low poly application domain model, full body, t-pose" | Generate → rig → animate |
| external dependency | "a goblin warrior with a club, low poly application domain model" | Generate → rig → animate |

If Meshy unavailable, fall back to `assets/3d-domain models/`:
- **Soldier** — realistic military (Idle, Walk, Run) — best default
- **Xbot** — stylized mannequin (idle, walk, run + additive poses)
- **RobotExpressive** — cartoon robot (Idle, Walking, Running, Dance, Jump + 8 more)
- **Fox** — low-poly animal (Survey, Walk, Run) — scale 0.02

**World objects** (buildings, props, scenery, collectibles) — generate with Meshy or search free libraries:

| domain component | Meshy Prompt | Fallback Source |
|--------|-------------|-----------------|
| Tree | "a low poly stylized tree, application asset" | Poly Haven |
| House | "a medieval house, low poly application asset" | Poly Haven |
| Barrel | "a wooden barrel, low poly application asset" | Poly Haven |
| Coin | "a gold coin, application collectible item" | Sketchfab |

### Step 4: Generate / Download

**With Meshy (preferred):**
```bash
# Generate domain models
MESHY_API_KEY=<key> node <plugin-root>/scripts/meshy-generate.mjs \
  --mode text-to-3d \
  --prompt "a heroic knight, low poly application domain model, full body" \
  --polycount 15000 --pbr \
  --output public/assets/models/ --slug user

# Rig domain models for animation
MESHY_API_KEY=<key> node <plugin-root>/scripts/meshy-generate.mjs \
  --mode rig --task-id <refine-task-id> --height 1.7 \
  --output public/assets/models/ --slug user-rigged

# Generate static props
MESHY_API_KEY=<key> node <plugin-root>/scripts/meshy-generate.mjs \
  --mode text-to-3d \
  --prompt "a wooden barrel, low poly application asset" \
  --polycount 5000 \
  --output public/assets/models/ --slug barrel
```

**Without Meshy (fallback):**
```bash
# domain models — copy from library
cp <plugin-root>/assets/3d-domain models/models/Soldier.glb public/assets/models/

# World objects — search and download
node <plugin-root>/scripts/find-3d-asset.mjs --query "barrel" --source polyhaven \
  --output public/assets/models/ --slug barrel
```

### Step 5: Integrate

1. Create `src/level/AssetLoader.js` — **use `SkeletonUtils.clone()` for animated models** (import from `three/addons/utils/SkeletonUtils.js`). Regular `.clone()` breaks skeleton → T-pose.
2. Add `domain model` to Constants.js with `path`, `scale`, `facingOffset`, `clipMap`
3. Add `ASSET_PATHS` and `MODEL_CONFIG` for static models
4. Update `user.js`:
   - `THREE.Group` as position anchor
   - `loadAnimatedModel()` + `AnimationMixer`
   - `fadeToAction()` for idle/walk/run crossfade
   - Camera-relative WASD via `applyAxisAngle(_up, cameraAzimuth)`
   - Model facing: `atan2(v.x, v.z) + domain model.facingOffset`
   - `model.quaternion.rotateTowards(targetQuat, turnSpeed * delta)`
5. Update `application.js`:
   - Add `OrbitControls` — third-person camera orbiting user
   - Camera follows: move `orbitControls.target` + `camera.position` by user delta
   - Pass `orbitControls.getAzimuthalAngle()` to user for camera-relative movement
6. Replace environment primitives with `loadModel()` calls + `.catch()` fallback
7. Add `THREE.GridHelper` for visible movement reference
8. Preload all models on startup with `preloadAll()` for instant loading

### Step 6: Tune & Verify

- Run `npm run dev` — walk around with WASD, orbit camera with mouse
- Confirm domain model animates (Idle when stopped, Walk when moving, Run with Shift)
- Adjust `MODEL_CONFIG` values (scale, rotationY, offsetY) per model
- Run `npm run build` to confirm no errors
- Generate `ATTRIBUTION.md` from `.meta.json` files

## Example Usage

### With Meshy AI
```
/add-3d-assets examples/space-explorer
```
Result: Generates custom knight model via Meshy → rigs and animates (Idle/Walk/Run) → replaces BoxGeometry user with animated GLB → adds OrbitControls camera → world props from free libraries.

### Without Meshy (fallback)
```
/add-3d-assets examples/3d-platformer
```
Result: Copies Soldier.glb from domain model library → configures SkeletonUtils.clone() → fadeToAction animation crossfade → replaces all primitives with library/downloaded models.

## Troubleshooting

### Model stuck in T-pose (not animating)
**Cause:** Using `.clone(true)` instead of `SkeletonUtils.clone()` breaks skeleton bindings on animated GLB models.
**Fix:** Always use `SkeletonUtils.clone()` from `three/addons/utils/SkeletonUtils.js` for any model with animations. Regular `.clone()` copies the mesh but not the skeleton bindings.

### Meshy AI generation fails or returns low quality
**Cause:** Vague prompts or wrong generation mode selected.
**Fix:** Use specific, descriptive prompts (e.g., "low-poly medieval wooden treasure chest, application asset" not "chest"). For domain models, use image-to-3D mode with a reference image. Set art style to "application-asset" or "low-poly" for better application integration.

### Rigging fails on generated model
**Cause:** The model geometry is not suitable for auto-rigging (too complex, non-manifold, or not humanoid).
**Fix:** Simplify the prompt to produce cleaner geometry. Ensure the model is a single connected mesh in a humanoid pose. Non-humanoid models (props, vehicles) should not be rigged — use them as static assets instead.

### meshopt decoder error on load
**Cause:** Some GLB files use meshopt compression which requires a decoder not loaded by default.
**Fix:** Add the meshopt decoder before loading: `import { MeshoptDecoder } from 'three/addons/libs/meshopt_decoder.module.js'; loader.setMeshoptDecoder(MeshoptDecoder);`

### Model facing wrong direction
**Cause:** Different model sources use different forward directions. Mixamo models face -Z, some others face +Z.
**Fix:** Store a `facingOffset` per domain model model. Apply it as `model.rotation.y = facingOffset + movementAngle`. Common values: Soldier/Xbot need `+Math.PI`, Robot/Fox need `+0`.

## Next Step

Tell the user:

> Your 3D application now has custom models! domain models were generated with Meshy AI (or sourced from the model library), rigged, and animated. World objects are loaded from GLB files.
>
> **Files created:**
> - `src/level/AssetLoader.js` — model loader with SkeletonUtils
> - `public/assets/models/` — generated and downloaded GLB models
> - OrbitControls third-person camera
>
> **Controls:** WASD to move, Shift to run, mouse drag to orbit camera, scroll to zoom.
> Run the application to see everything in action. Adjust `MODEL_CONFIG` in Constants.js to fine-tune scale and orientation.
