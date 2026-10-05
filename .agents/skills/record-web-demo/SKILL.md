---
name: record-web-demo
description: Capture a repeatable Playwright video or screenshot sequence of a React web application for QA, documentation, or stakeholder review. Use when the user asks to record a browser workflow.
argument-hint: "[workflow]"
license: MIT
compatibility: Requires Playwright.
metadata:
  tags: [web, demo, screenshots, playwright]
---

# Record Web Demo

Create a deterministic capture recipe.

- Seed the data and use a dedicated test account or fixture.
- Set viewport, color scheme, locale, reduced-motion preference, and network behavior explicitly.
- Capture the initial state, key interaction, and final result with sensitive fields masked.
- Wait on semantic readiness signals instead of arbitrary delays.
- Review for clipped text, focus visibility, loading artifacts, console errors, and responsive failures.

## Full Workflow

1. Choose the workflow, seed data, authentication fixture, target browsers, and exact viewport dimensions.
2. Configure locale, color scheme, reduced motion, network behavior, and deterministic time/randomness.
3. Capture via stable semantic readiness signals rather than arbitrary delays.
4. Include the start state, core interaction, success result, and recovery path where useful.
5. Mask sensitive fields and inspect screenshots/video for overlap, clipping, focus, loading artifacts, and console errors.
6. Store the reproducible capture recipe and output metadata with the milestone or release record.