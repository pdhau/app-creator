---
name: make-app
description: Use this skill to design, scaffold, build, test, and iterate on a web, desktop, mobile, CLI, or service application. The default browser stack is React with Vite or Next.js.
metadata:
  author: application-creator
  version: "3.1"
  tags: [application, react, nextjs, vite, workflow, planning, testing, delivery]
---

# Make App

Use this skill as the main application workflow. Keep the project explicit: idea, scaffold, then development.

## Rules

- Do not create implementation files before the approved `docs/product-plan.md`, `docs/tech.md`, and first ADR exist.
- In an existing repository, run `sub-pipelines/session-start.md` first.
- Keep one focused feature per session. Run scope triage when the request contains independent work; record deferred work in `docs/backlog.md`.
- Ground substantial work in a milestone with observable acceptance criteria and an exit condition.
- Write failing acceptance tests before implementation. For React applications, use Playwright DOM/API assertions, accessible roles, deterministic mocks, and focused unit tests.
- After every code change, run `sub-pipelines/live-iterate.md`: check console, routes, loading/error/empty/success states, screenshots where relevant, and adjacent workflows.
- Apply `vercel-react-best-practices` when touching React or Next.js code, `vercel-composition-patterns` for component API design, and `web-design-guidelines` for UI/accessibility review.
- Update `docs/STATE.md`, milestone status, ADRs, and handoff notes whenever the repository changes.
- Prefer official framework scaffolders. Do not hand-write generated project configuration when an official initializer exists.

## Phases

### Idea

Run `phase-pipelines/idea.md`. Clarify users, problem, primary workflow, success and failure states, scope, constraints, risks, stack, testing, and deployment. Produce `docs/product-plan.md`, `docs/tech.md`, and `docs/architectural-decisions/0001-*.md`.

### Scaffold

Run `phase-pipelines/scaffold.md`. Initialize React with Vite or Next.js unless the product explicitly requires another runtime, install dependencies, establish the project layout, add testability hooks, boot the first route/view, and verify a clean console. Then run `sub-pipelines/agents-bootstrap.md` and create `docs/STATE.md`.

### Development

Run `phase-pipelines/development.md` for every feature or refactor:

1. Read `docs/STATE.md`, `docs/product-plan.md`, `docs/tech.md`, the active milestone, and backlog.
2. Triage scope and update documentation before coding when scope changes.
3. Write and run a failing test for each checkable acceptance criterion.
4. Implement the smallest correct vertical slice.
5. Run live verification and the focused regression suite.
6. Update milestone acceptance criteria and `docs/STATE.md`.

## Stack Defaults

- **Standard web application:** React + Vite + TypeScript.
- **Full-stack or SEO-sensitive application:** Next.js + TypeScript.
- **Mobile application:** React Native + Expo, with `vercel-react-native-skills`.
- **3D or spatial surface:** React + Vite/Next.js plus `threejs-app`; do not use a game engine by default.
- **Deployment:** Vercel is the default web target; use `deploy-to-vercel`, `vercel-cli-with-tokens`, and `vercel-optimize` as needed.

## Source-of-Truth Files

- `docs/product-plan.md`: users, problem, workflow, outcomes, constraints, and anti-goals.
- `docs/tech.md`: framework, language, dependencies, commands, conventions, and deployment.
- `docs/milestones/*.md`: ordered objectives, scope, acceptance criteria, and exit conditions.
- `docs/backlog.md`: append-only deferred ideas and open questions.
- `docs/STATE.md`: current phase, last action, next step, and blockers.
- `docs/architectural-decisions/*.md`: durable technical and product architecture decisions.
- `AGENTS.md`: cross-tool enforcement generated from the source-of-truth docs.
