---
name: react-canvas
description: Integrate an interactive 2D canvas into a React web application for editors, diagrams, maps, charts, or visual tools. Use when the user needs canvas rendering or pointer interactions inside React.
argument-hint: "[canvas experience]"
license: MIT
metadata:
  tags: [react, canvas, interaction, visualization]
---

# React Canvas

Keep the React UI and the imperative canvas renderer at a clear boundary.

- Store domain state outside the draw loop and derive a render model from it.
- Use `useRef` for the canvas and imperative handles; initialize and dispose listeners in effects.
- Support pointer, touch, keyboard alternatives, zoom, resize, high-DPI rendering, and reduced motion.
- Keep hit testing deterministic and expose accessible labels or a structured fallback for important content.
- Throttle expensive redraws and avoid recreating contexts, assets, or event handlers unnecessarily.
- Test resize, cleanup, keyboard operation, and a representative interaction.

## Full Workflow

1. Define the canvas task, domain model, coordinate system, gestures, keyboard alternative, and fallback representation.
2. Inspect existing rendering, state, resize, accessibility, and asset conventions.
3. Write tests for hit testing, resize, cleanup, and the main interaction.
4. Implement the imperative renderer behind a stable React boundary; avoid recreating contexts and assets on render.
5. Add pointer, touch, keyboard, zoom, high-DPI, reduced-motion, loading, and error behavior.
6. Verify performance, focus, mobile gestures, route unmount, and fallback content in production preview.
7. Record the coordinate/input contract and rendering limits in the milestone.