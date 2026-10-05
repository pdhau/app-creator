---
name: web-3d-assets
description: Plan a production 3D asset pipeline for React web applications, including formats, optimization, loading, caching, and accessible fallbacks. Use when a web product contains many 3D assets or a reusable 3D viewer.
argument-hint: "[asset pipeline]"
license: MIT
metadata:
  tags: [react, 3d, pipeline, performance]
---

# Web 3D Assets

Define an asset contract before adding files.

- Standardize format, scale, coordinate system, naming, license metadata, and texture budgets.
- Preprocess and compress assets in CI or a repeatable build step.
- Lazy-load by route or viewport and cache immutable files with content hashes.
- Track load time, memory, GPU cost, and failure fallback on target devices.
- Keep an HTML summary or alternate image for essential product information.
- Remove unused assets and verify paths from the production preview.

## Full Workflow

1. Define the 3D asset contract: format, scale, coordinate system, naming, license metadata, texture budget, and fallback.
2. Audit source assets, preprocessing tools, route boundaries, CDN/cache behavior, and target device limits.
3. Add a repeatable conversion/compression step and tests for invalid, missing, oversized, and unavailable assets.
4. Integrate lazy loading, progress, error recovery, disposal, and an accessible HTML/image alternative.
5. Measure cold load, cache hit, memory, GPU cost, frame time, and route-change cleanup.
6. Verify production paths on desktop/mobile and document the asset inventory, budget, and known limitations.