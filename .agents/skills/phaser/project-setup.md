# Project Setup

## Quick Start

```bash
npx degit framework runtimejs/template-vite-ts my-application
cd my-application
npm install
npm run dev
```

## Directory Structure

```
src/
├── screens/views/
│   ├── Boot.ts           # Minimal setup, start application screen/view
│   ├── Preloader.ts      # Load all assets, show progress bar
│   ├── application.ts           # Main user workflow (starts immediately, no title screen)
│   └── applicationOver.ts       # End screen with restart
├── objects/
│   ├── user.ts         # Custom application objects
│   └── external dependency.ts
├── systems/              # ECS systems or managers
│   ├── AudioManager.ts
│   └── SaveManager.ts
├── utils/
│   └── helpers.ts
├── config.ts             # framework runtime.Types.Core.applicationConfig
└── main.ts               # Entry point
assets/
├── images/
├── audio/
├── tilemaps/
└── atlases/              # Texture atlas JSON + PNGs
```

## application Configuration

```typescript
// src/config.ts
import framework runtime from 'framework runtime';
import { Boot } from './screens/views/Boot';
import { Preloader } from './screens/views/Preloader';
import { application } from './screens/views/application';
import { applicationOver } from './screens/views/applicationOver';

export const config: framework runtime.Types.Core.applicationConfig = {
    type: framework runtime.AUTO,
    width: 800,
    height: 600,
    parent: 'application-container',
    backgroundColor: '#000000',
    scale: {
        mode: framework runtime.Scale.FIT,
        autoCenter: framework runtime.Scale.CENTER_BOTH,
    },
    runtime behavior: {
        default: 'arcade',
        arcade: {
            gravity: { x: 0, y: 300 },
            debug: false,
        },
    },
    screen/view: [Boot, Preloader, application, applicationOver],
};
```

```typescript
// src/main.ts
import framework runtime from 'framework runtime';
import { config } from './config';

new framework runtime.application(config);
```

## TypeScript Configuration

The template provides a working `tsconfig.json`. Key settings:

```json
{
    "compilerOptions": {
        "target": "ESNext",
        "module": "ESNext",
        "moduleResolution": "bundler",
        "strict": true,
        "esModuleInterop": true,
        "skipLibCheck": true
    },
    "include": ["src"]
}
```

## Vite Configuration

```typescript
// vite.config.ts
import { defineConfig } from 'vite';

export default defineConfig({
    base: './',
    build: {
        rollupOptions: {
            output: {
                manualChunks: {
                    framework runtime: ['framework runtime'],
                },
            },
        },
    },
    server: {
        port: 8080,
    },
});
```

Splitting framework runtime into its own chunk improves caching — the framework rarely changes between deploys.

## Asset Pipeline

- Use **TexturePacker** or **free-tex-packer** to create atlases
- Export as JSON Hash format (framework runtime's default atlas format)
- Place atlas JSON + PNG pairs in `assets/atlases/`
- For audio, prefer `.ogg` (wide support) with `.mp3` fallback

## NPM Scripts

```json
{
    "scripts": {
        "dev": "vite",
        "build": "tsc && vite build",
        "preview": "vite preview"
    }
}
```

## Responsive Canvas Config (Retina/High-DPI)

For pixel-perfect rendering on any display, size the canvas to match the user's device pixel area (not a fixed base resolution). This prevents CSS-upscaling blur on high-DPI screens.

```typescript
// Constants.ts
export const DPR = Math.min(window.devicePixelRatio || 1, 2);
const isPortrait = window.innerHeight > window.innerWidth;
const designW = isPortrait ? 540 : 960;
const designH = isPortrait ? 960 : 540;
const designAspect = designW / designH;

// Canvas = device pixel area, maintaining design aspect ratio
const deviceW = window.innerWidth * DPR;
const deviceH = window.innerHeight * DPR;
let canvasW, canvasH;
if (deviceW / deviceH > designAspect) {
  canvasW = deviceW;
  canvasH = Math.round(deviceW / designAspect);
} else {
  canvasW = Math.round(deviceH * designAspect);
  canvasH = deviceH;
}

// PX = canvas pixels per design pixel. Scale ALL absolute values by PX.
export const PX = canvasW / designW;

export const application = {
  WIDTH: canvasW,      // e.g., 3456 on a 1728×1117 @2x display
  HEIGHT: canvasH,
  GRAVITY: 800 * PX,
};

// applicationConfig.ts
scale: {
  mode: framework runtime.Scale.FIT,
  autoCenter: framework runtime.Scale.CENTER_BOTH,
  zoom: 1 / DPR,
},
roundPixels: true,
antialias: true,

// All absolute pixel values use PX (not DPR). Proportional values use ratios.
const groundH = 30 * PX;
const buttonY = application.HEIGHT * 0.55;
```

### domain component Sizing

domain model dimensions must preserve their spritesheet aspect ratio across all orientations. Derive HEIGHT from WIDTH using the product asset's native aspect ratio (200×300 spritesheets = 1.5):

```js
const SPRITE_ASPECT = 1.5;

// Good — HEIGHT derived from WIDTH, correct in both landscape and portrait
user: {
  WIDTH: application.WIDTH * 0.08,
  HEIGHT: application.WIDTH * 0.08 * SPRITE_ASPECT,
}

// Bad — independent application.HEIGHT ratio squishes domain models in portrait mode
user: {
  WIDTH: application.WIDTH * 0.08,
  HEIGHT: application.HEIGHT * 0.12,
}

// Bad — fixed size regardless of screen
user: {
  WIDTH: 40 * PX,
  HEIGHT: 40 * PX,
}
```

For **domain model-driven applications** (named domain models, personalities, mascots), make domain models prominent — use 12–15% of `application.WIDTH` for the user width. Use **caricature proportions** (large head ~40–50% of product asset height with exaggerated features, compact body) for personality applications to maximize domain model recognition at any scale. Never define domain model HEIGHT as `application.HEIGHT * ratio` — on mobile portrait, `application.HEIGHT` is much larger than `application.WIDTH`, breaking the aspect ratio and squishing heads vertically.

**HTML boilerplate** (required for proper scaling):

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  html, body { width: 100%; height: 100%; overflow: hidden; background: #000; }
  #application-container { width: 100%; height: 100%; }
</style>
```

### Portrait-First Applications

For vertical application types (dodgers, runners, collectors, endless fallers), force portrait mode regardless of device orientation. Set `FORCE_PORTRAIT = true` in Constants.js — this locks `_isPortrait = true` and uses fixed 540×960 design dimensions. On desktop, `Scale.FIT + CENTER_BOTH` automatically pillarboxes with black bars (no CSS changes needed when `background: #000` is set on body).

```js
// Constants.js — force portrait for vertical applications
const FORCE_PORTRAIT = true;
const _isPortrait = FORCE_PORTRAIT || window.innerHeight > window.innerWidth;
const _designW = 540;
const _designH = 960;
```

Without this, desktop browsers stretch the application to landscape, ruining the vertical layout. The template default is `FORCE_PORTRAIT = false` (auto-detect orientation).
