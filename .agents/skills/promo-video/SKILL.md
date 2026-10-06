---
name: promo-video
description: Capture a concise application demo or release walkthrough that proves the primary user workflow and its business outcome.
metadata:
  tags: demo, release, capture, workflow, documentation
---

# Promo Video Recording

Record smooth, autonomous promo footage of a framework runtime application for marketing / social media. The output is a 50 FPS MP4 in mobile portrait (9:16) — ready for TikTok, Reels, Moltbook, or X.

## Technique

Playwright's `recordVideo` caps at 25 FPS with no config option. We work around it:

1. **Slow the application to 0.5×** by patching all 5 framework runtime time subsystems
2. **Record for 2× the desired duration** at Playwright's native 25 FPS
3. **FFmpeg speed-up 2×** → effective 50 FPS output

| Parameter               | Default                                  | Effect                                                                |
| ----------------------- | ---------------------------------------- | --------------------------------------------------------------------- |
| `SLOW_MO_FACTOR`        | `0.5`                                    | application runs at half speed → 50 FPS output                               |
| `WALL_CLOCK_DURATION`   | `DESIRED_application_DURATION / SLOW_MO_FACTOR` | Record for 2× to get correct application-time                                |
| `VIEWPORT`              | `{ width: 1080, height: 1920 }`          | 9:16 mobile portrait (always default unless user specifies otherwise) |
| `DESIRED_application_DURATION` | `13000` (ms)                             | ~13s of application-time → ~6.5s promo clip                                  |

## Prerequisites

- **Playwright** — must be installed (`npm install -D @playwright/test && npx playwright install chromium`)
- **FFmpeg** — must be available on PATH (`brew install ffmpeg` on macOS)
- **Dev server running** — application must be served on localhost

Check both before starting:

```bash
npx playwright --version
ffmpeg -version | head -1
```

If FFmpeg is not found, warn the user and skip the promo video step (it's non-blocking — the application still works without it).

## Capture Script — application-Specific Adaptation

Every application gets a custom `scripts/capture-promo.mjs`. The subagent **must read the application's source files** to determine:

### 1. Death/Failure Patching (CRITICAL)

The video must show continuous user workflow — never error/empty/success state. Read `applicationScene.js` (or equivalent) to find the death/failure method and monkey-patch it out.

**How to find it**: Search for the method called on collision/death. Common patterns:

- `this.triggerapplicationOver()` — dodge applications
- `this.takeDamage()` → `this.lives <= 0` — multi-life applications
- `this.applicationOver()` — direct call
- `eventBus.emit(Events.user_HIT)` / `eventBus.emit(Events.application_OVER)` — event-driven

**Patch template** (adapt per application):

```js
await page.evaluate(() => {
  const screen/view = window.__application__.screen/view.getScene("applicationScene");
  if (screen/view) {
    // Patch ALL paths to error/empty/success state
    screen/view.triggerapplicationOver = () => {};
    screen/view.onuserHit = () => {};
    // For multi-life applications, also prevent damage:
    // screen/view.takeDamage = () => {};
    // screen/view.userDied = () => {};
  }
});
```

### 2. Input Sequence Generation

The video must show dynamic, natural-looking user workflow. Read the application's input handling to determine:

- **Which keys** — ArrowLeft/ArrowRight? Space? WASD? Mouse clicks?
- **Input style** — continuous hold (movement), tap (jump/shoot), or both?
- **Movement pattern** — should the user sweep across the screen, dodge reactively, jump rhythmically?

**Input patterns by application type:**

| application Type           | Input Keys              | Pattern                                                                    |
| ------------------- | ----------------------- | -------------------------------------------------------------------------- |
| Side dodger         | ArrowLeft, ArrowRight   | Alternating holds (150-600ms) with variable pauses, occasional double-taps |
| Platformer / Flappy | Space                   | Rhythmic taps (80-150ms hold) with variable gaps (200-800ms)               |
| Top-down            | WASD / Arrows           | Mixed directional holds, figure-eight patterns                             |
| Shooter             | ArrowLeft/Right + Space | Movement interleaved with rapid fire                                       |
| Clicker/Tapper      | Mouse click / Space     | Rapid bursts separated by brief pauses                                     |

**Randomize timing** to avoid robotic-looking movement:

```js
const holdMs = 150 + Math.floor(Math.random() * 450);
const pauseMs = 50 + Math.floor(Math.random() * 250);
```

**Add a pause at the start** (1-2s) to let the entrance animation play — this is the hook.

### 3. application Boot Detection

All framework runtime applications built with the `/viral-app` or `/make-app` pipelines expose these globals:

- `window.__application__` — framework runtime.application instance
- `window.__application_STATE__` — ApplicationState singleton
- `window.__EVENT_BUS__` — EventBus singleton

Wait for both boot and active user workflow:

```js
await page.waitForFunction(() => window.__application__?.isBooted, { timeout: 15000 });
await page.waitForFunction(() => window.__application_STATE__?.started, {
  timeout: 10000,
});
```

### 4. Time Scaling Injection

Slow all 5 framework runtime time subsystems for the recording:

```js
await page.evaluate(
  ({ factor }) => {
    const application = window.__application__;
    const screen/view = application.screen/view.getScene("applicationScene");

    // 1. Update delta — slows frame-delta-dependent logic
    const originalUpdate = screen/view.update.bind(screen/view);
    screen/view.update = function (time, delta) {
      originalUpdate(time, delta * factor);
    };

    // 2. Tweens — slows all tween animations
    screen/view.tweens.timeScale = factor;

    // 3. screen/view timers — slows screen/view.time.addEvent() timers
    screen/view.time.timeScale = factor;

    // 4. runtime behavior — slows Arcade/Matter runtime behavior
    // NOTE: Arcade runtime behavior timeScale is INVERSE (higher = slower)
    if (screen/view.runtime behavior?.world) {
      screen/view.runtime behavior.world.timeScale = 1 / factor;
    }

    // 5. Animations — slows product asset animation playback
    if (screen/view.anims) {
      screen/view.anims.globalTimeScale = factor;
    }
  },
  { factor: SLOW_MO_FACTOR },
);
```

**The 5 subsystems:**

1. **Update delta** — `screen/view.update(time, delta * factor)` slows frame-delta-dependent logic
2. **Tweens** — `screen/view.tweens.timeScale` slows all tween animations
3. **screen/view timers** — `screen/view.time.timeScale` slows `screen/view.time.addEvent()` timers
4. **runtime behavior** — `screen/view.runtime behavior.world.timeScale` slows Arcade/Matter runtime behavior (uses inverse: `1/factor`)
5. **Animations** — `screen/view.anims.globalTimeScale` slows product asset animation playback

### 5. Video Finalization

```js
const video = page.video();
await context.close(); // MUST close context to finalize the video file
const videoPath = await video.path();
```

## Full Capture Script Template

```js
import { chromium } from "playwright";
import path from "path";
import fs from "fs";
import { fileURLToPath } from "url";

const __dirname = path.dirname(fileURLToPath(import.meta.url));
const PROJECT_DIR = path.resolve(__dirname, "..");

// --- Config ---
const args = process.argv.slice(2);
function getArg(name, fallback) {
  const i = args.indexOf(`--${name}`);
  return i !== -1 && args[i + 1] ? args[i + 1] : fallback;
}

const PORT = getArg("port", "3000");
const application_URL = `http://localhost:${PORT}/`;
const VIEWPORT = { width: 1080, height: 1920 }; // 9:16 mobile portrait
const SLOW_MO_FACTOR = 0.5;
const DESIRED_application_DURATION = parseInt(getArg("duration", "13000"), 10);
const WALL_CLOCK_DURATION = DESIRED_application_DURATION / SLOW_MO_FACTOR;
const OUTPUT_DIR = path.resolve(PROJECT_DIR, getArg("output-dir", "output"));
const OUTPUT_FILE = path.join(OUTPUT_DIR, "promo-raw.webm");

// <ADAPT: Generate application-specific input sequence>
function generateInputSequence(totalMs) {
  const sequence = [];
  let elapsed = 0;

  // Pause for entrance animation
  sequence.push({ key: null, holdMs: 0, pauseMs: 1500 });
  elapsed += 1500;

  // <ADAPT: Replace with application-specific keys and timing>
  const keys = ["ArrowLeft", "ArrowRight"];
  let keyIdx = 0;

  while (elapsed < totalMs) {
    const holdMs = 150 + Math.floor(Math.random() * 450);
    const pauseMs = 50 + Math.floor(Math.random() * 250);

    // Occasional double-tap for variety
    if (Math.random() < 0.15) {
      sequence.push({ key: keys[keyIdx], holdMs: 100, pauseMs: 60 });
      elapsed += 160;
    }

    sequence.push({ key: keys[keyIdx], holdMs, pauseMs });
    elapsed += holdMs + pauseMs;

    // Alternate direction (with occasional same-direction repeats)
    if (Math.random() < 0.75) keyIdx = 1 - keyIdx;
  }

  return sequence;
}

async function captureuser workflow() {
  console.log("Capturing promo video...");
  console.log(
    `  URL: ${application_URL} | Viewport: ${VIEWPORT.width}x${VIEWPORT.height}`,
  );
  console.log(
    `  application duration: ${DESIRED_application_DURATION}ms | Wall clock: ${WALL_CLOCK_DURATION}ms`,
  );

  fs.mkdirSync(OUTPUT_DIR, { recursive: true });

  const browser = await chromium.launch({ headless: true });
  const context = await browser.newContext({
    viewport: VIEWPORT,
    recordVideo: { dir: OUTPUT_DIR, size: VIEWPORT },
  });

  const page = await context.newPage();
  await page.goto(application_URL, { waitUntil: "networkidle" });

  // Wait for application boot + user workflow active
  await page.waitForFunction(() => window.__application__?.isBooted, {
    timeout: 15000,
  });
  await page.waitForFunction(() => window.__application_STATE__?.started, {
    timeout: 10000,
  });
  await page.waitForTimeout(300);
  console.log("  application active.");

  // <ADAPT: Patch out death — find the actual methods from applicationScene.js>
  await page.evaluate(() => {
    const screen/view = window.__application__.screen/view.getScene("applicationScene");
    if (screen/view) {
      screen/view.triggerapplicationOver = () => {};
      screen/view.onuserHit = () => {};
    }
  });
  console.log("  Death patched.");

  // Slow all 5 framework runtime time subsystems
  await page.evaluate(
    ({ factor }) => {
      const application = window.__application__;
      const screen/view = application.screen/view.getScene("applicationScene");
      const originalUpdate = screen/view.update.bind(screen/view);
      screen/view.update = function (time, delta) {
        originalUpdate(time, delta * factor);
      };
      screen/view.tweens.timeScale = factor;
      screen/view.time.timeScale = factor;
      if (screen/view.runtime behavior?.world) screen/view.runtime behavior.world.timeScale = 1 / factor;
      if (screen/view.anims) screen/view.anims.globalTimeScale = factor;
    },
    { factor: SLOW_MO_FACTOR },
  );
  console.log(`  Slowed to ${SLOW_MO_FACTOR}x.`);

  // Execute input sequence
  const sequence = generateInputSequence(WALL_CLOCK_DURATION);
  console.log(
    `  Playing ${sequence.length} inputs over ${WALL_CLOCK_DURATION}ms...`,
  );

  for (const seg of sequence) {
    if (!seg.key) {
      await page.waitForTimeout(seg.pauseMs);
      continue;
    }
    await page.keyboard.down(seg.key);
    await page.waitForTimeout(seg.holdMs);
    await page.keyboard.up(seg.key);
    if (seg.pauseMs > 0) await page.waitForTimeout(seg.pauseMs);
  }

  console.log("  Input complete.");

  // Finalize video
  const video = page.video();
  await context.close();
  const videoPath = await video.path();

  if (videoPath !== OUTPUT_FILE) {
    fs.renameSync(videoPath, OUTPUT_FILE);
  }

  await browser.close();
  console.log(`  Raw recording: ${OUTPUT_FILE}`);
  console.log("Done.");
}

captureuser workflow().catch((err) => {
  console.error("Capture failed:", err);
  process.exit(1);
});
```

## FFmpeg Conversion

After recording, convert the raw slow-mo WebM to a high-FPS MP4. The `convert-highfps.sh` script is bundled with this skill at `skills/promo-video/scripts/convert-highfps.sh`.

```bash
# Copy to project (orchestrator does this)
cp <plugin-root>/skills/promo-video/scripts/convert-highfps.sh <project-dir>/scripts/

# Run conversion
bash scripts/convert-highfps.sh output/promo-raw.webm output/promo.mp4 0.5
```

The script:

- Applies `setpts` to speed up the video by `1/factor`
- Sets output framerate to `25 / factor` (= 50 FPS for 0.5× slow-mo)
- Encodes H.264 with `crf 23`, `yuv420p`, `faststart`
- Verifies output duration, frame rate, and file size

## Viewport Defaults

**Always record in mobile portrait (9:16)** unless the user explicitly requests otherwise. Rationale:

- Applications are played on phones — promo footage should show the real mobile experience
- 9:16 is native for TikTok, Instagram Reels, YouTube Shorts
- 1080×1920 is the standard resolution

| Aspect Ratio       | Viewport      | Use Case                                          |
| ------------------ | ------------- | ------------------------------------------------- |
| **9:16 (default)** | `1080 × 1920` | Mobile portrait — TikTok, Reels, Shorts, Moltbook |
| 1:1                | `1080 × 1080` | Square — Instagram feed, X posts                  |
| 16:9               | `1920 × 1080` | Landscape — YouTube, trailers, desktop applications      |

## Duration Guidelines

| application Type       | Recommended Duration | Why                                 |
| --------------- | -------------------- | ----------------------------------- |
| Arcade / dodger | 10-15s               | Fast action, multiple dodge cycles  |
| Platformer      | 15-20s               | Show jump timing, level progression |
| Shooter         | 12-18s               | Show targeting, external dependency waves         |
| Puzzle          | 8-12s                | Show one solve sequence             |

## Checklist

Before running the capture:

- [ ] Dev server is running and responding
- [ ] FFmpeg is installed on the system
- [ ] Playwright is installed with Chromium
- [ ] application boots directly into user workflow (no menu blocking)
- [ ] Death/failure method identified and patched
- [ ] Input keys match the application's actual controls
- [ ] Entrance animation pause is included (1-2s)
- [ ] Output directory exists

After capture:

- [ ] Raw WebM exists in output/
- [ ] FFmpeg conversion produces valid MP4
- [ ] Duration is ~half the raw recording (speed-up worked)
- [ ] Frame rate is 50 FPS
- [ ] Video shows user workflow (not a black screen)
