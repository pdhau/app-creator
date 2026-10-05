---
name: review-react-app
description: Review a React web application for correctness, architecture, performance, accessibility, security, and test gaps. Use when the user asks for a code review or application audit. This skill is read-only.
argument-hint: "[area or path]"
license: MIT
metadata:
  tags: [react, review, security, performance, accessibility]
---

# Review React App

Prioritize actionable findings over style commentary.

- Trace the requested workflow from route to UI, state, API, and persistence.
- Look for broken states, race conditions, missing cleanup, unsafe input, authorization gaps, and responsive regressions.
- Check bundle and render cost, data fetching, caching, error boundaries, and secret handling.
- Check semantic HTML, keyboard access, focus management, contrast, labels, and reduced motion.
- Verify tests cover the risky behavior and distinguish confirmed findings from open questions.
- Report findings first, ordered by severity, with file references and a concise residual-risk summary.

## Full Workflow

1. Read the requested scope, product plan, technical plan, active milestone, and test/build scripts.
2. Trace the user workflow from route through components, state/query, API, persistence, and authorization.
3. Reproduce suspicious behavior and inspect diagnostics before forming a finding.
4. Review correctness, race conditions, cleanup, input validation, authorization, secrets, performance, accessibility, and tests.
5. Rank confirmed findings by severity and include a concrete fix direction and file reference.
6. Run focused checks when available; separate confirmed issues, open questions, and residual risk.
7. Keep the review read-only unless the user explicitly asks to implement fixes.