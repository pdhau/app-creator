---
name: webgl-performance
description: Diagnose and improve WebGL or 3D rendering performance inside a React web application. Use when a 3D view has slow frames, high memory, long loads, or excessive GPU work.
argument-hint: "[performance issue]"
license: MIT
metadata:
  tags: [react, webgl, threejs, performance]
---

# WebGL Performance

Measure before changing rendering code.

- Capture frame time, memory, draw calls, texture memory, shader cost, and load timing on target devices.
- Reduce work with lazy loading, visibility culling, instancing where valid, compressed textures, and lower device pixel ratio.
- Avoid rebuilding scene objects or materials on every React render.
- Dispose geometries, materials, textures, and render targets when routes change.
- Preserve readable labels and a non-WebGL fallback for essential information.
- Re-test cold load, route changes, resize, low-power devices, and production builds.

## Full Workflow

1. Capture a reproducible slow scenario with route, viewport, device, data, and network conditions.
2. Measure frame time, memory, draw calls, texture memory, shader cost, load timing, and React render frequency.
3. Set a budget and write a regression smoke check for the worst user-visible path.
4. Fix the highest-cost root cause using lazy loading, culling, compression, stable objects, or lower pixel ratio.
5. Dispose resources and verify route changes, context loss, resize, and fallback information.
6. Compare before/after measurements on target desktop and mobile devices.
7. Record residual risk and the next measurable optimization in the milestone or backlog.