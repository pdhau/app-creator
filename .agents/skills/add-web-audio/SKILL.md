---
name: add-web-audio
description: Add intentional audio feedback, narration, or media playback to a React web application. Use when the user asks for web audio, sound effects, voice, music, or media controls.
argument-hint: "[audio behavior]"
license: MIT
metadata:
  tags: [react, audio, media, accessibility]
---

# Add Web Audio

Audio must be optional, user-controlled, and compatible with browser autoplay policies.

- Use the existing media or audio library when present.
- Expose play, pause, mute, volume, and loading/error states with accessible labels.
- Start playback only after a user gesture unless the browser permits otherwise.
- Respect reduced motion, OS mute, data usage, and focus changes.
- Stop and dispose listeners when components unmount.
- Test blocked autoplay, slow media, mute persistence, keyboard controls, and mobile layout.

## Full Workflow

1. Define why audio exists, the user control model, supported formats, privacy expectations, and fallback content.
2. Inspect existing media utilities and browser support before adding Web Audio or another dependency.
3. Model idle, loading, playing, paused, muted, blocked, ended, and failed states.
4. Write tests for user-gesture playback, mute/volume, autoplay rejection, keyboard control, unmount cleanup, and persistence.
5. Implement the smallest accessible control surface and keep audio resources outside render paths.
6. Test mobile browsers, hidden documents, reduced-motion preferences, slow media, and route changes.
7. Document licensing, preload policy, bandwidth impact, and verification evidence.