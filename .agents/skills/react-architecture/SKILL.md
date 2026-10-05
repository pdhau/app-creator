---
name: react-architecture
description: Design or refactor architecture for a maintainable React web application. Use when deciding component boundaries, state management, routing, services, data fetching, configuration, or project structure.
argument-hint: "[architecture question]"
license: MIT
metadata:
  tags: [react, architecture, typescript, state-management]
---

# React Architecture

Choose boundaries that follow product domains and data ownership.

- Keep route composition, feature logic, reusable UI, services, and configuration distinct.
- Use local state for local interaction, a store for shared client state, and a query layer for server state.
- Keep API clients typed and validate untrusted responses at the boundary.
- Use events only for cross-cutting notifications, not as a replacement for React state.
- Clean up subscriptions, timers, requests, and observers on unmount.
- Avoid circular dependencies, barrel-file surprises, speculative abstractions, and client-side secrets.
- Record major decisions in a short ADR when the repository uses them.

## Full Workflow

1. Read product plan, technical plan, current state, active milestone, and nearest route/feature code.
2. Map route composition, feature domains, reusable UI, server state, client state, services, configuration, and persistence.
3. Identify the smallest boundary change that resolves the requirement and list migration risks.
4. Write an ADR when changing stack, state ownership, API contracts, or top-level module boundaries.
5. Add tests for state transitions, API validation, cleanup, and the affected user workflow.
6. Implement incrementally, run typecheck and focused tests after each boundary change, then verify the browser flow.
7. Update `docs/tech.md`, milestone evidence, `docs/STATE.md`, and backlog for deferred migrations.