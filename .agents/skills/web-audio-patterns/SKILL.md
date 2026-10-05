---
name: web-audio-patterns
description: Apply reusable browser audio and media patterns in a React web application, including player state, autoplay policy, mute persistence, and accessible controls. Use when audio is a real product requirement.
argument-hint: "[audio pattern]"
license: MIT
metadata:
  tags: [react, audio, media, patterns]
---

# Web Audio Patterns

Model media as explicit state: idle, loading, playing, paused, muted, blocked, ended, and failed.

- Keep the media element or AudioContext behind a small hook/service boundary.
- Let users start playback and always provide mute and volume controls.
- Persist preferences only with consent and handle multiple tabs carefully.
- Pause or reduce work when the document is hidden or the user prefers reduced motion.
- Clean up event listeners and object URLs.
- Test browser autoplay rejection, device changes, keyboard controls, and unmount cleanup.

## Full Workflow

1. Define media purpose, supported browsers, user consent, controls, persistence, and fallback content.
2. Inspect the existing media element, hook, state, and analytics conventions before adding abstractions.
3. Write tests for idle/loading/playing/paused/muted/blocked/ended/failed transitions and user-gesture requirements.
4. Implement the smallest service/hook boundary with cleanup, hidden-document behavior, and reduced-motion support.
5. Add accessible controls and verify slow media, autoplay rejection, device changes, keyboard use, and route unmount.
6. Check bandwidth, preload, licensing, and mobile behavior in production preview.
7. Document the state contract and unresolved browser limitations.