---
name: add-react-feature
description: Add a complete feature to a React web application, including user flow, state, API integration, loading and error states, accessibility, tests, and documentation. Use when the user asks to add a feature, workflow, page, form, dashboard, or interaction.
argument-hint: "[feature or workflow]"
license: MIT
metadata:
  tags: [react, feature, typescript, product]
---

# Add React Feature

Implement the smallest vertical slice that delivers the requested user outcome.

1. Inspect the existing routes, component conventions, state/query layer, API client, and test setup.
2. Define acceptance criteria for happy, loading, empty, error, permission, offline, and mobile states.
3. Add the feature near its owning route or domain. Keep server state in the existing query layer and local UI state in the component or store.
4. Use semantic HTML, keyboard support, visible focus, stable test IDs only where needed, and abortable async work.
5. Add focused tests for the primary workflow and validation failure, then run the narrowest available check.

Do not invent a new state library, design system, or API contract when the application already has one.

## Full Workflow

1. Read project state, product plan, active milestone, backlog, route map, and nearest feature implementation.
2. Convert the request into one user outcome and explicit acceptance criteria for success, loading, empty, error, permission, offline, keyboard, and responsive states.
3. Trace the owning route, component boundary, client/server state, API contract, validation, and existing design primitives.
4. Write failing tests for each checkable criterion before implementation. Use unit tests for pure logic and browser tests for user-visible behavior.
5. Implement the smallest vertical slice, preserving existing public contracts and keeping async work abortable.
6. Run browser live iteration after every code change: console, network, primary flow, boundary states, focus, and mobile viewport.
7. Run focused tests, typecheck/lint, and the nearest production build or preview check.
8. Update the milestone evidence, `docs/STATE.md`, and `docs/backlog.md` for deferred requests.

## Completion

Do not call the feature complete from compilation alone. The exit condition must be observable by a user and supported by executable evidence or an explicit user-verified visual check.