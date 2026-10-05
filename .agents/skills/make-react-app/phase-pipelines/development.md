# Development Phase Pipeline

Apply this pipeline to every feature, refactor, and user-visible bug fix.

1. Run `sub-pipelines/session-start.md` and read the active milestone and backlog.
2. Run `sub-pipelines/scope-triage.md` if the request contains independent work.
3. Create or update a milestone with objective, scope, acceptance criteria, dependencies, and exit condition.
4. Turn each checkable acceptance criterion into a failing test before implementation.
5. Trace the owning route, feature, state/query boundary, API contract, and existing component patterns.
6. Implement the smallest vertical slice, including loading, empty, error, permission, keyboard, mobile, and offline states where relevant.
7. Run `sub-pipelines/live-iterate.md` after every code change.
8. Run focused tests, typecheck/lint, production build, and preview smoke test when the slice is complete.
9. Update milestone evidence, `docs/STATE.md`, and append deferred items to `docs/backlog.md`.

Never loosen a failing test merely to make it pass. Change implementation, or explicitly revise the acceptance criterion with the user.