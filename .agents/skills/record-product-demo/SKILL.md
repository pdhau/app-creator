---
name: record-product-demo
description: Record a stable browser walkthrough or product demo for a React web application. Use when the user asks for a demo video, walkthrough, release clip, or UI capture.
argument-hint: "[route or workflow]"
license: MIT
compatibility: Requires Playwright; FFmpeg is optional for encoding.
metadata:
  tags: [react, demo, playwright, recording]
---

# Record Product Demo

Capture the user workflow, not a cinematic approximation of it.

- Define the route, seeded data, viewport, authentication fixture, and success outcome.
- Wait for stable UI state and disable nonessential animation or time-dependent content.
- Use accessible interactions so the recording also proves the workflow is operable.
- Keep sensitive data out of frames and verify the output at desktop and mobile aspect ratios.
- Encode only after the browser capture is correct; report the output path and prerequisites.

## Full Workflow

1. Select one user outcome and define route, fixture, account, viewport, locale, color scheme, and reduced-motion settings.
2. Prepare deterministic data and mask secrets or personal information before recording.
3. Write the capture steps as accessible browser actions and wait for semantic readiness signals.
4. Capture initial state, key interaction, result, and any meaningful error or recovery state.
5. Review for clipped text, focus visibility, console errors, loading artifacts, and responsive failures.
6. Encode only the verified capture and record the command, output, fixture, and known limitations.