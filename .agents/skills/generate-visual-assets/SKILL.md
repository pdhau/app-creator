---
name: generate-visual-assets
description: Generate or prepare visual assets for a React web application, including icons, illustrations, backgrounds, and product imagery. Use when the user asks for generated visual assets or a web asset pack.
argument-hint: "[asset brief]"
license: MIT
metadata:
  tags: [react, design, assets, imagery]
---

# Generate Visual Assets

Create assets that reinforce the product's information hierarchy and brand.

- Specify dimensions, aspect ratio, contrast, focal point, responsive crops, and licensing.
- Prefer real product imagery when users need to inspect an object or place.
- Export efficient formats with meaningful names, alt text guidance, and dark/light variants where needed.
- Keep decorative imagery separate from essential information and provide fallbacks.
- Check loading behavior, sharpness, contrast, and layout stability on mobile and desktop.

## Full Workflow

1. Read the product plan and define the asset's job, audience, crop, dimensions, contrast, and responsive variants.
2. Confirm licensing, source metadata, brand constraints, and whether the asset is essential or decorative.
3. Generate or prepare a representative asset and review it against the actual component context.
4. Export efficient formats, meaningful filenames, alt-text guidance, dark/light variants, and fallback content.
5. Add a visual or rendering check for the target route before wiring the final asset.
6. Verify sharpness, contrast, layout stability, loading, error fallback, and mobile/desktop crops.
7. Update the asset inventory and record unresolved art/content decisions in the backlog.