---
name: web-qa
description: Test React web applications with Playwright, accessibility checks, visual regression, performance smoke tests, and workflow fixtures. Use when reviewing or validating browser behavior.
argument-hint: "[workflow or risk]"
license: MIT
compatibility: Requires Node.js and Playwright.
metadata:
  tags: [react, web, qa, playwright, accessibility]
---

# Web QA

Test the product as a user experiences it.

- Configure `webServer`, `baseURL`, deterministic fixtures, and desktop/mobile projects.
- Prefer role, label, and text locators; use test IDs only for stable non-semantic targets.
- Cover boot, routes, forms, validation, async states, authorization, keyboard navigation, and responsive layout.
- Use axe on important routes and mask dynamic content in screenshots.
- Avoid arbitrary sleeps; wait for semantic readiness and network contracts.
- Report flaky tests, environment prerequisites, and untested risk explicitly.

## Full Workflow

1. Read product plan, active milestone, fixtures, scripts, and risk areas before writing tests.
2. Convert acceptance criteria into user-observable checks and define deterministic auth, data, viewport, and network setup.
3. Write failing tests first using role, label, and text locators; avoid coupling to component internals.
4. Cover boot, navigation, forms, validation, async states, authorization, keyboard, mobile, retry, and route changes.
5. Add accessibility checks and visual snapshots only for stable states; mask dynamic content and disable irrelevant motion.
6. Run narrow tests, then the relevant suite and production preview; investigate flakes instead of hiding them with sleeps.
7. Record coverage gaps, prerequisites, evidence, and follow-up defects in the milestone/backlog.