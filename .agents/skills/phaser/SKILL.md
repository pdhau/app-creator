---
name: phaser
description: Compatibility alias for application runtime patterns. Use React with Vite or Next.js for standard applications; use this alias only when an existing repository still references the legacy phaser path.
metadata:
  tags: [compatibility, application, react, runtime, routing, state, forms]
---

# Application Runtime Compatibility Alias

This directory is retained so existing references do not break. New application work should use React with Vite or Next.js through `make-app`.

## Default React Stack

- React + TypeScript for the UI and feature modules.
- Vite for a client-rendered application or Next.js for full-stack/SEO-sensitive work.
- Semantic HTML and accessible roles as the primary test selectors.
- Explicit loading, error, empty, success, and pending states.
- API and network adapters isolated from components.
- Apply `vercel-react-best-practices`, `vercel-composition-patterns`, and `web-design-guidelines` when relevant.

## Existing Repository Compatibility

If the repository already uses Phaser, preserve its existing architecture while migrating incrementally. Do not introduce new Phaser-specific code into a React-first application. Keep legacy scenes isolated behind an adapter and move new user workflows into React components and feature modules.

For a new project, follow:

- `../make-app/SKILL.md`
- `../make-app/phase-pipelines/scaffold.md`
- `../make-app/phase-pipelines/development.md`
- `../vercel-react-best-practices/SKILL.md`

The canonical runtime skill is `make-app`; this alias exists only for compatibility.
