---
name: ship-react-app
description: Take a React web application from validated idea to production release with focused implementation, testing, deployment, and handoff. Use for an end-to-end build or release workflow.
argument-hint: "[app or release goal]"
license: MIT
metadata:
  tags: [react, release, deploy, product]
---

# Ship React App

Use a short loop: understand, implement, verify, release.

1. Inspect the repository, scripts, environment, routes, and deployment target.
2. Define the primary user outcome, acceptance criteria, data contract, and smallest milestone.
3. Implement the vertical slice with responsive and accessible states.
4. Run focused tests, typecheck/lint, production build, and a preview smoke test.
5. Check environment variables, asset paths, source maps, redirects, headers, and error reporting.
6. Record the release notes, known limitations, rollback path, and next task.

Never put secrets in `VITE_*` variables or ship unverified placeholder content.

## Full Workflow

1. Read project state, product plan, technical plan, active milestone, backlog, scripts, and environment contract.
2. Define one release outcome, acceptance criteria, dependencies, non-goals, and rollback path.
3. Implement the vertical slice with explicit loading, empty, error, permission, offline, responsive, and keyboard states.
4. Write failing tests first, then run live browser iteration after every code change.
5. Run focused tests, full relevant suite, typecheck/lint, build, preview smoke test, and deployment checks.
6. Validate assets, routes, redirects, headers, API origins, monitoring, and production console/network behavior.
7. Release through the existing pipeline, record evidence and known limitations, and update state/backlog for the next session.