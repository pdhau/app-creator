---
name: add-web-3d-assets
description: Add optimized 3D models and scenes to a React web application when 3D is part of the product experience. Use for GLB or glTF assets, model previews, product configurators, and Three.js integrations.
argument-hint: "[asset or scene]"
license: MIT
metadata:
  tags: [react, 3d, glb, gltf, assets]
---

# Add Web 3D Assets

Use 3D only when it helps users inspect, configure, or understand the product.

- Inspect the current renderer and asset pipeline before adding a dependency.
- Prefer compressed GLB, lazy loading, responsive framing, and a useful loading fallback.
- Keep model loading in a component boundary with cancellation, error recovery, and disposal on unmount.
- Provide keyboard and non-3D alternatives when the scene conveys essential information.
- Check bundle size, mobile GPU cost, texture dimensions, and production asset paths.
- Verify the scene in desktop and mobile browsers.

## Full Workflow

1. Confirm that 3D improves inspection or configuration and define an accessible non-3D fallback.
2. Audit the existing renderer, asset conventions, route boundaries, bundle budget, and target devices.
3. Specify format, scale, camera framing, materials, license metadata, loading states, and failure behavior.
4. Prepare a compressed asset and write a failing smoke test for route load, unavailable asset, resize, and unmount cleanup.
5. Integrate lazy loading behind an error boundary; keep model state separate from React route state.
6. Verify memory, frame time, texture cost, keyboard access, reduced motion, mobile GPU behavior, and production URLs.
7. Update asset inventory, milestone evidence, and the next concrete handoff step.