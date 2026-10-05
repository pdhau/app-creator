---
name: add-web-qa
description: Add focused Playwright QA coverage to a React web application. Use when the user asks to add tests, visual regression, accessibility checks, smoke tests, or browser coverage.
argument-hint: "[workflow or risk]"
license: MIT
compatibility: Requires Node.js and Playwright.
metadata:
  tags: [react, qa, playwright, accessibility, visual-regression]
---

# Add Web QA

Add tests around user-visible behavior, not implementation details.

- Reuse the project's `webServer`, `baseURL`, fixtures, and browser projects.
- Prefer accessible locators and stable user-facing contracts.
- Cover boot, navigation, primary interaction, validation, loading, error, empty, and permission states.
- Add axe checks for important routes and visual snapshots only after the UI is deterministic.
- Mask timestamps, random IDs, network-dependent content, and animations in screenshots.
- Run the focused test locally and report any environment prerequisite.

## Full Workflow

1. Read the active milestone and convert each acceptance criterion into a browser-observable check.
2. Inspect test scripts, fixtures, `webServer`, `baseURL`, authentication setup, browser projects, and CI limits.
3. Write the failing test first, using roles, labels, and stable user contracts rather than implementation selectors.
4. Add deterministic fixtures and control time, random data, animation, and network boundaries.
5. Cover primary workflows plus validation, loading, empty, error, permission, keyboard, mobile, and retry behavior.
6. Add axe and visual assertions only for stable states; mask dynamic content.
7. Run the narrow test, then the relevant suite, and record flaky or environment-dependent gaps.