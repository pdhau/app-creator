---
name: make-react-app
description: Plan, scaffold, build, or iterate on a production-minded React web application. Use for a new app, product concept, route, architecture, milestone plan, or end-to-end implementation.
argument-hint: "[application idea or milestone]"
license: MIT
metadata:
  tags: [react, vite, typescript, architecture, product]
---

# Make React App

Use this skill as the complete lifecycle workflow for a React web product: idea, scaffold, development, verification, release, and cross-session handoff. Follow the phase pipeline that matches the repository state instead of improvising a shorter process.

## Operating Rules

- Read `docs/STATE.md` first in an existing project, then `docs/product-plan.md`, `docs/tech.md`, open milestones, and `docs/backlog.md` when present.
- Inspect the repository before choosing tools. Prefer the existing React, TypeScript, Vite, routing, state, query, and test conventions.
- Keep each session focused on one feature or one coherent change. Run `sub-pipelines/scope-triage.md` when the request contains independent asks.
- Nothing mentioned by the user is silently dropped. Defer out-of-scope work to `docs/backlog.md` using `templates/backlog.md`.
- Write failing tests before implementation for checkable acceptance criteria. Use Playwright for user workflows, unit tests for pure logic, and contract tests for API boundaries.
- Run the live iteration loop after every code change: inspect console, exercise the workflow, verify state, and capture a screenshot when visual behavior changed.
- Update product documentation and milestone acceptance criteria as implementation progresses. A feature is not complete until its user-visible exit condition is verified.
- Keep secrets server-side, validate external data at boundaries, and include loading, empty, error, permission, offline, keyboard, and responsive states.

## Phase Routing

Determine the phase from the repository and documentation, then follow its pipeline:

1. **Idea**: the product outcome, users, workflows, constraints, and architecture are not yet agreed.
2. **Scaffold**: the stack is chosen but the application does not yet boot with a verified first route.
3. **Development**: the app boots and work is organized into milestones.

If documentation is missing in a project that has code, run `sub-pipelines/session-start.md` and backfill the minimum source-of-truth files before feature work.

## Idea Phase

Follow [phase-pipelines/idea.md](phase-pipelines/idea.md). Complete this phase when:

- `docs/` exists.
- `docs/product-plan.md` describes users, product goal, primary workflows, scope, and acceptance language.
- `docs/tech.md` records the React stack and the reason for each important dependency.
- `docs/architectural-decisions/0001-stack-and-boundaries.md` records the initial stack and boundary decisions.
- The user has confirmed the primary workflow and the next phase.

## Scaffold Phase

Follow [phase-pipelines/scaffold.md](phase-pipelines/scaffold.md). Prefer official scaffolding commands and current framework documentation. Complete this phase only when:

- Initial application files and dependencies exist.
- The app boots in a browser with an initial route and no console errors.
- The production build and preview work.
- `AGENTS.md` and `CLAUDE.md` exist at the project root.
- A future agent can run the project with one documented command.

Compilation alone is insufficient; the browser smoke test is mandatory.

## Development Phase

Follow [phase-pipelines/development.md](phase-pipelines/development.md) for every feature, refactor, and user-visible fix. Ground large work in `docs/milestones/*.md`. A milestone must contain:

- Objective and user outcome.
- Scope and explicit non-goals.
- Ordered dependencies.
- Checkable acceptance criteria.
- Exit condition written as `User does X -> observes Y`.
- Verification evidence and follow-up backlog entries.

Order architecture-enabling work before product polish. Append in-scope refinements to the current milestone; create a new milestone for a separate system or feature area; use the backlog for small isolated follow-ups.

## Documentation Contract

The following files are the continuity system for the project:

- `docs/product-plan.md`: product goal, users, workflows, scope, and behavior.
- `docs/tech.md`: stack, scripts, data boundaries, runtime configuration, and architecture constraints.
- `docs/milestones/*.md`: delivery units and acceptance evidence.
- `docs/architectural-decisions/*.md`: detailed decisions and their reasons.
- `docs/backlog.md`: append-only home for deferred or rejected ideas.
- `docs/STATE.md`: last completed action, active milestone, blockers, and next concrete step.
- `AGENTS.md`: generated enforcement reflection for other coding tools.
- `CLAUDE.md`: one-line pointer to `AGENTS.md`.

Read each document at the times described by [sub-pipelines/session-start.md](sub-pipelines/session-start.md), and update `docs/STATE.md` at the end of every session that changes code, docs, or decisions.

If the user asks to skip documentation, use minimum-viable doc mode: one-line milestone, mandatory `STATE.md`, mandatory `product-plan.md` and `tech.md`, and ADRs for stack or top-level architecture changes. Compress documentation; do not delete the continuity contract.

## Sub-pipelines

Before inventing a process, check [sub-pipelines/README.md](sub-pipelines/README.md). Available focused workflows include session start, scope triage, milestone planning, user-flow reproduction, live iteration, bug fixing, asset pipeline work, state-machine design, documentation drift audits, and agent bootstrap.

Use [useful-skills.md](useful-skills.md) to select neighboring React skills such as UI design, feature work, QA, deployment, assets, realtime, and entitlements.

## Completion Checklist

- The requested user outcome works in the browser.
- Acceptance criteria have executable evidence or are explicitly marked user-verified when inherently visual.
- Focused tests, typecheck/lint, production build, and preview smoke test have run where available.
- Responsive, accessibility, loading, error, empty, permission, and offline behavior were considered.
- Milestone docs and `docs/STATE.md` are current.
- Deferred work is recorded in `docs/backlog.md`.