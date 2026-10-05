---
name: design-react-ui
description: Design or improve a polished, responsive, accessible React interface. Use when the user asks to improve UI, UX, visual design, responsive behavior, forms, states, or interaction quality.
argument-hint: "[screen or experience]"
license: MIT
metadata:
  tags: [react, ui, ux, responsive, accessibility]
---

# Design React UI

Start from the user task and the existing visual language.

- Inspect current tokens, typography, layout primitives, icon library, and breakpoint conventions.
- Define hierarchy, primary action, navigation, content density, and responsive behavior before styling.
- Design loading, empty, error, disabled, validation, success, and offline states.
- Use semantic landmarks, accessible names, 44px touch targets, keyboard flow, visible focus, and reduced-motion support.
- Keep sections unframed unless a card genuinely represents a repeated item, modal, or tool.
- Verify the rendered result at narrow mobile and wide desktop sizes.

## Full Workflow

1. Read the product plan and inspect the current tokens, typography, component library, routes, and brand constraints.
2. Map the information hierarchy, primary action, navigation, permissions, content density, and responsive breakpoints.
3. Specify every meaningful state before styling: loading, empty, error, disabled, validation, success, and offline.
4. Implement semantic structure and layout primitives before decorative polish.
5. Add purposeful motion only where it explains change, with reduced-motion behavior and no layout instability.
6. Validate keyboard flow, focus visibility, labels, contrast, touch targets, and mobile/desktop screenshots.
7. Record design decisions and unresolved content or interaction questions in the active milestone.