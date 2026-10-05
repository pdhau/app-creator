---
name: quick-react-prototype
description: Rapidly scaffold a small React prototype to validate a web product idea or workflow. Use when the user wants a fast prototype, proof of concept, or clickable interaction.
argument-hint: "[idea or workflow]"
license: MIT
metadata:
  tags: [react, prototype, vite, fast]
---

# Quick React Prototype

Optimize for learning while preserving a clean upgrade path.

- Use the repository's existing toolchain; otherwise prefer Vite, React, and TypeScript.
- Build one realistic workflow with representative data, not a pile of placeholder screens.
- Include navigation, primary interaction, validation, loading, empty, error, and mobile states.
- Keep mock data behind a small adapter so a real API can replace it.
- Use semantic controls and keyboard support from the first pass.
- Run the app, test the primary path, and record deliberate prototype limitations.

## Full Workflow

1. Clarify the single hypothesis and user workflow the prototype must validate.
2. Inspect repository conventions; use the official React scaffolder only when no app exists.
3. Create representative mock data and an adapter boundary for future API replacement.
4. Write a small smoke check for boot, navigation, primary interaction, and failure state.
5. Build the workflow with semantic controls, realistic content, responsive layout, and explicit limitations.
6. Run live browser iteration and collect the user's decision or feedback, not just a screenshot.
7. Record what is disposable, what is production-ready, and the next milestone.