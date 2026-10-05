---
name: use-react-template
description: Start a React web application from an existing template or repository pattern. Use when the user asks to use a template, scaffold a Vite app, or bootstrap a standard React project.
argument-hint: "[template or app type]"
license: MIT
metadata:
  tags: [react, vite, template, scaffold]
---

# Use React Template

Choose the smallest template that matches the requested product.

- Inspect available repository templates and required versions before cloning or generating.
- Prefer React + TypeScript + Vite unless the product needs an existing framework convention.
- Preserve package manager, scripts, linting, test setup, and environment conventions.
- Remove sample routes and assets after confirming the app boots, then add the first real workflow.
- Verify install, development server, production build, preview, and a basic browser smoke test.
- Document any template-specific commands or assumptions.

## Full Workflow

1. Identify the required framework, React version, package manager, routing, styling, testing, and deploy target.
2. Inspect candidate templates and official setup documentation; choose the smallest compatible one.
3. Scaffold with the official command and verify install, dev server, test, build, and preview before customization.
4. Remove sample routes/assets only after confirming the baseline boot path and preserve useful configuration.
5. Add the first real route with loading, error, empty, accessibility, and mobile states.
6. Run a browser smoke test and record commands, version assumptions, and migration risks in `docs/tech.md`.