---
name: generate-3d-environments
description: Generate an optional 3D environment for a React web product such as a virtual tour, showroom, map, or immersive landing view. Use when the user requests a generated 3D scene or environment.
argument-hint: "[environment brief]"
license: MIT
compatibility: Requires the configured environment provider credentials and internet access.
metadata:
  tags: [react, 3d, environment, immersive-web]
---

# Generate 3D Environments

An environment must support a clear product task and have a lightweight fallback.

- Define the user's goal, navigation model, camera bounds, accessibility alternative, and mobile budget.
- Generate only approved source material and record license/provider metadata.
- Stream or lazy-load large scene data; never block the whole app on an optional scene.
- Provide loading, unavailable, reduced-motion, keyboard, and low-power states.
- Test memory, frame rate, asset paths, and behavior after route changes.

## Full Workflow

1. Define the product task, navigation model, scene bounds, target devices, and HTML alternative.
2. Inspect existing 3D boundaries, asset delivery, route loading, analytics, and reduced-motion conventions.
3. Generate a representative scene and validate provider rights, scale, materials, and spatial orientation.
4. Establish loading, unavailable, low-power, keyboard, reduced-motion, and route-cancel behavior.
5. Lazy-load the scene behind a feature boundary and keep essential information outside WebGL.
6. Verify memory, frame rate, context loss, resize, route changes, and production asset URLs.
7. Document provider metadata, performance measurements, fallback behavior, and next iteration.