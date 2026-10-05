---
name: generate-3d-assets
description: Generate or prepare custom 3D assets for a React web application, then optimize and integrate them into an existing 3D preview or configurator. Use when the user asks for a custom model or generated 3D asset.
argument-hint: "[asset brief]"
license: MIT
compatibility: Requires the chosen asset provider credentials and internet access.
metadata:
  tags: [react, 3d, assets, generation]
---

# Generate 3D Assets

Treat generated media as a product asset pipeline, not as an application secret.

1. Clarify dimensions, camera views, materials, licensing, and target device budget.
2. Generate or obtain the asset through the configured provider.
3. Validate geometry, textures, orientation, scale, naming, and license metadata.
4. Convert and compress to the web's supported format, preferably GLB with appropriate texture compression.
5. Integrate with lazy loading, fallback UI, cache headers, and an accessible alternative.
6. Verify production URLs and mobile performance.

## Full Workflow

1. Write the asset brief: purpose, dimensions, camera views, materials, target devices, license, and fallback.
2. Confirm provider credentials are server-side and define a reproducible output location and metadata record.
3. Generate or obtain the model, then validate scale, orientation, topology, textures, naming, and license.
4. Optimize to the web budget and write a smoke check for load, failure, and route cleanup.
5. Integrate through the existing lazy-loading and 3D viewer boundary, never directly into unrelated UI.
6. Verify cold load, cache hit, memory, mobile GPU cost, production path, and accessible alternative.
7. Record the source, transformation steps, known limitations, and handoff.