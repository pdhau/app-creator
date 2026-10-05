---
name: improve-react-app
description: Audit and implement the highest-impact improvements to an existing React web application. Use when the user asks to improve, fix, polish, speed up, or stabilize an application.
argument-hint: "[area to improve]"
license: MIT
metadata:
  tags: [react, improve, performance, accessibility, reliability]
---

# Improve React App

Work from evidence and fix one coherent slice at a time.

1. Reproduce the issue or inspect the requested area and existing diagnostics.
2. Rank findings by user impact, correctness, accessibility, performance, and regression risk.
3. Make the smallest root-cause change consistent with the current architecture.
4. Cover the changed behavior with a focused test or executable check.
5. Recheck responsive rendering, console errors, loading states, and production build behavior.

Do not introduce broad refactors or replace established libraries without a concrete reason.

## Full Workflow

1. Read `docs/STATE.md`, active milestones, backlog, scripts, diagnostics, and the requested user journey.
2. Reproduce the issue or measure the suspected performance/accessibility problem before editing.
3. Rank findings by correctness, user impact, accessibility, security, performance, and regression risk.
4. Write the smallest failing regression test or executable check that captures the root cause.
5. Implement one coherent fix and run live browser iteration immediately.
6. Re-run focused tests, typecheck/lint, and the nearest broader check; inspect console and production preview.
7. Update documentation, milestone evidence, state handoff, and deferred backlog items.