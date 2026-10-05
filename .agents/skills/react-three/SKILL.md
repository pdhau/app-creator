---
name: react-three
description: Add a Three.js or React Three Fiber view to a React web application for product visualization, maps, data, or immersive interfaces. Use when the user explicitly needs WebGL or 3D.
argument-hint: "[3D feature]"
license: MIT
metadata:
  tags: [react, threejs, react-three-fiber, webgl]
---

# React Three

Use WebGL for a concrete product need and preserve a usable non-WebGL path.

- Keep scene setup, loading, controls, and disposal inside the 3D feature boundary.
- Lazy-load the renderer and assets; provide loading, error, low-power, keyboard, and reduced-motion states.
- Limit draw calls, texture memory, post-processing, and animation work on mobile.
- Keep important information available as semantic HTML or an accessible summary.
- Test route changes, resize, context loss, asset failure, and production builds.
- Follow the repository's existing Three.js and React version conventions.

## Full Workflow

1. Confirm WebGL is necessary and define the semantic HTML fallback, camera task, controls, and target device budget.
2. Inspect renderer versions, asset pipeline, route boundaries, lazy loading, error boundaries, and performance telemetry.
3. Write smoke checks for initial load, asset failure, resize, route change, context loss, and disposal.
4. Implement the scene behind a feature boundary with loading, low-power, keyboard, reduced-motion, and error states.
5. Keep scene objects and materials stable across React renders; dispose resources on unmount.
6. Measure cold load, frame time, memory, draw calls, texture cost, and mobile behavior.
7. Update technical docs and milestone evidence with fallback and performance results.