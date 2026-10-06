---
name: record-promo
description: Record an application release demo for onboarding, review, support, or stakeholder communication.
metadata:
  tags: recording, demo, onboarding, release
---

# Record Release Demo

Use a deterministic environment and a short script: boot, key workflow, measurable outcome, and recovery path. Confirm the recording contains no secrets or personal data and link it from the release handoff.---
name: record-promo
description: Record an autonomous 50 FPS promo video of your application for social media. Use when the user says "record a video", "make a promo", "capture user workflow", "record user workflow footage", or "make a trailer for my application". Do NOT use for user workflow testing (use qa-app).
argument-hint: "[path-to-application]"
license: MIT
metadata:
  author: OpusapplicationLabs
  version: 1.3.0
  tags: [application, video, promo, recording, playwright, ffmpeg]
---

# Record Promo Video

Autonomously capture smooth, high-FPS user workflow footage of your framework runtime application — ready for TikTok, Reels, Moltbook, or X. No human input needed during recording.

## Instructions

Record a promo video of the application at `$ARGUMENTS` (or the current directory if no path given).

### 1. Pre-checks

Verify these are available:
- **Dev server**: Check if one is already running (`curl -s http://localhost:3000/`). If not, start one with `npm run dev` in the background. Note the port.
- **Playwright**: `npx playwright --version`. If not installed: `npm install -D @playwright/test && npx playwright install chromium`
- **FFmpeg**: `ffmpeg -version | head -1`. If not found, tell the user to install it (`brew install ffmpeg` on macOS) and stop.

### 2. Generate the capture script

Load the `promo-video` skill for the full technique reference.

Read the application's source files to understand:
- `src/screens/views/applicationScene.js` — find death/failure methods to patch out
- `src/core/EventBus.js` — event flow
- `src/core/Constants.js` — input keys, application dimensions
- `src/main.js` — verify `__application__` and `__application_STATE__` globals are exposed

Create `scripts/capture-promo.mjs` adapted for this specific application:
- Patch out ALL death/failure code paths
- Generate input sequences matching the application's actual controls
- Include entrance animation pause (1-2s)
- Default viewport: `1080 × 1920` (9:16 mobile portrait)
- Default duration: 13s application-time

### 3. Copy the conversion script

```bash
cp <plugin-root>/skills/promo-video/scripts/convert-highfps.sh <project-dir>/scripts/
chmod +x <project-dir>/scripts/convert-highfps.sh
```

### 4. Run the capture

```bash
mkdir -p output
node scripts/capture-promo.mjs --port <port>
bash scripts/convert-highfps.sh output/promo-raw.webm output/promo.mp4 0.5
```

### 5. Verify and preview

Extract a thumbnail at the 5-second mark and show it to the user:
```bash
ffmpeg -y -ss 5 -i output/promo.mp4 -frames:v 1 -update 1 output/promo-thumbnail.jpg
```

Read the thumbnail image and display it. Report duration, frame rate, and file size.

## Example Usage

```
/record-promo examples/flappy-bird
```
Result: Patches out death → generates input sequence for the application's controls → captures 13s of autonomous user workflow at 1080x1920 (9:16) → converts to 50 FPS MP4 → `output/promo.mp4` ready for TikTok/Reels.

Tell the user:
> Promo video recorded! 50 FPS, mobile portrait (1080×1920).
>
> **File**: `output/promo.mp4`
>
> Post it to TikTok, Reels, or X to drive traffic to your application.
