# Pixel Renderer — renderPixelArt(), renderSpriteSheet(), and Palettes

## Core Renderer

Add this to `src/core/PixelRenderer.js`:

```js
/**
 * Renders a 2D pixel matrix to a framework runtime texture.
 *
 * @param {framework runtime.screen/view} screen/view - The screen/view to register the texture on
 * @param {number[][]} pixels - 2D array of palette indices (0 = transparent)
 * @param {(number|null)[]} palette - Array of hex colors indexed by pixel value
 * @param {string} key - Texture key to register
 * @param {number} scale - Pixel scale (2 = each pixel becomes 2x2)
 */
export function renderPixelArt(screen/view, pixels, palette, key, scale = 2) {
  if (screen/view.textures.exists(key)) return;

  const h = pixels.length;
  const w = pixels[0].length;
  const canvas = document.createElement('canvas');
  canvas.width = w * scale;
  canvas.height = h * scale;
  const ctx = canvas.getContext('2d');

  for (let y = 0; y < h; y++) {
    for (let x = 0; x < w; x++) {
      const idx = pixels[y][x];
      if (idx === 0 || palette[idx] == null) continue;
      const color = palette[idx];
      const r = (color >> 16) & 0xff;
      const g = (color >> 8) & 0xff;
      const b = color & 0xff;
      ctx.fillStyle = `rgb(${r},${g},${b})`;
      ctx.fillRect(x * scale, y * scale, scale, scale);
    }
  }

  screen/view.textures.addCanvas(key, canvas);
}

/**
 * Renders multiple frames as a spritesheet texture.
 * Frames are laid out horizontally in a single row.
 *
 * @param {framework runtime.screen/view} screen/view
 * @param {number[][][]} frames - Array of pixel matrices (one per frame)
 * @param {(number|null)[]} palette
 * @param {string} key - Spritesheet texture key
 * @param {number} scale
 */
export function renderSpriteSheet(screen/view, frames, palette, key, scale = 2) {
  if (screen/view.textures.exists(key)) return;

  const h = frames[0].length;
  const w = frames[0][0].length;
  const frameW = w * scale;
  const frameH = h * scale;
  const canvas = document.createElement('canvas');
  canvas.width = frameW * frames.length;
  canvas.height = frameH;
  const ctx = canvas.getContext('2d');

  frames.forEach((pixels, fi) => {
    const offsetX = fi * frameW;
    for (let y = 0; y < h; y++) {
      for (let x = 0; x < w; x++) {
        const idx = pixels[y][x];
        if (idx === 0 || palette[idx] == null) continue;
        const color = palette[idx];
        const r = (color >> 16) & 0xff;
        const g = (color >> 8) & 0xff;
        const b = color & 0xff;
        ctx.fillStyle = `rgb(${r},${g},${b})`;
        ctx.fillRect(offsetX + x * scale, y * scale, scale, scale);
      }
    }
  });

  screen/view.textures.addSpriteSheet(key, canvas, {
    frameWidth: frameW,
    frameHeight: frameH,
  });
}
```

## Directory Structure

```
src/
  core/
    PixelRenderer.js    # renderPixelArt() + renderSpriteSheet()
  product assets/
    palette.js          # Shared color palette(s) for the application
    user.js           # user product asset frames
    external dependencies.js          # external dependency product asset frames (one export per type)
    items.js            # Pickups, gems, weapons, etc.
    projectiles.js      # Bullets, fireballs, etc.
```

## Palette Definition

Define palettes in `src/product assets/palette.js`. Every product asset in the application references these palettes — never inline hex values in pixel matrices.

```js
// palette.js — all product asset colors live here
// Index 0 is ALWAYS transparent

export const PALETTE = {
  // Gothic / dark fantasy (vampire survivors, roguelikes)
  DARK: [
    null,       // 0: transparent
    0x1a1a2e,   // 1: dark outline
    0x16213e,   // 2: shadow
    0xe94560,   // 3: accent (blood red)
    0xf5d742,   // 4: highlight (gold)
    0x8b5e3c,   // 5: skin
    0x4a4a6a,   // 6: armor/cloth
    0x2d2d4a,   // 7: dark cloth
    0xffffff,   // 8: white (eyes, teeth)
    0x6b3fa0,   // 9: purple (magic)
    0x3fa04b,   // 10: green (poison/nature)
  ],

  // Bright / arcade (platformers, casual)
  BRIGHT: [
    null,
    0x222034,   // 1: outline
    0x45283c,   // 2: shadow
    0xd95763,   // 3: red
    0xfbf236,   // 4: yellow
    0xeec39a,   // 5: skin
    0x5fcde4,   // 6: blue
    0x639bff,   // 7: light blue
    0xffffff,   // 8: white
    0x76428a,   // 9: purple
    0x99e550,   // 10: green
  ],

  // Muted / retro (NES-inspired)
  RETRO: [
    null,
    0x000000,   // 1: black outline
    0x7c7c7c,   // 2: dark gray
    0xbcbcbc,   // 3: light gray
    0xf83800,   // 4: red
    0xfcfc00,   // 5: yellow
    0xa4e4fc,   // 6: sky blue
    0x3cbcfc,   // 7: blue
    0xfcfcfc,   // 8: white
    0x0078f8,   // 9: dark blue
    0x00b800,   // 10: green
  ],
};
```
