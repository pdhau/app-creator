# AGENTS.md

> Lean enforcement file for any AI agent working in this repository. Read this file **first**, every session, before any other action. Cross-tool standard (Cursor, Aider, Codex, Claude, etc.). Claude users: `CLAUDE.md` points here.

This project follows the [`make-app`](https://github.com/OpusapplicationLabs/application-creator) pipeline. Do not improvise process — follow the rules below.

## Project overview

- **Title:** <application title>
- **Pitch:** <1–2 sentence pitch from `docs/product-plan.md`>
- **Genre / type:** <e.g. roguelike deckbuilder, 2.5D pixel-art blackjack>
- **framework runtime:** <framework runtime + version, from `docs/tech.md`>
- **Language:** <e.g. TypeScript, C#>
- **Target platform:** <e.g. Browser (desktop + mobile web)>

## Hard gate: no code before plan

If `docs/product-plan.md` does not exist in this repo, you are in the **idea phase** — do **not** create source files, run framework runtime scaffolders, or install dependencies. Your only job is to walk the user through the idea-phase clarifying-questions checklist (user workflow loop, art style, controls/scope, framework runtime/stack — see `make-app` skill `phase-pipelines/idea.md`) and produce `docs/product-plan.md`, `docs/tech.md`, and `docs/architectural-decisions/0001-framework runtime-and-stack.md`. A detailed-sounding initial prompt is **not** a substitute for the checklist; ask every item even if the user "seems clear."

## Mandatory: run the make-app skill

Every session in this directory **must** run the `make-app` skill's session-start sub-pipeline before any other work. The skill enforces phase awareness, doc continuity, milestone discipline, and the live-iterate verification loop.

If the skill is not installed in your environment:

```
npx skills add opusapplicationlabs/application-creator --skill make-app -p -y
```

If the skill cannot be installed (e.g. unsupported agent tool), apply the rules in this file manually and tell the user the skill is missing.

## Source-of-truth files

These docs are the agreed state of the project. Read them at session start; do **not** unilaterally edit them — propose changes and get user confirmation first.

- `docs/STATE.md` — last session handoff. Read first, every session.
- `docs/product-plan.md` — application definition (loop, rules, art, audio, anti-goals).
- `docs/tech.md` — stack, tooling, conventions.
- `docs/milestones/` — feature work breakdown with acceptance criteria.
- `docs/architectural-decisions/` — locked top-level decisions (framework runtime, language, art-style, state-machine pattern, etc.).

## Architecture rules

<Filled in per-stack at scaffold time. Examples:>

<For browser applications (framework runtime 3 / rich visual runtime):>
- **EventBus singleton** — all cross-module communication via pub/sub. Modules never import each other directly. Events use `domain:action` naming.
- **ApplicationState singleton** — single centralized state object. Systems read; events trigger mutations.
- **Constants.js** — every magic number, color, timing, speed lives here. Zero hardcoded values in application logic.
- **`window.render_application_to_text()`** — exposes a JSON snapshot of current application state for agent inspection without screenshots.
- **`window.advanceTime(ms)`** — steps the simulation deterministically for verification.

<For Unity / Unreal / Godot: replace with framework runtime-appropriate patterns.>

## Stack-specific commands

- **Dev server:** `<command>`
- **Tests:** `<command>`
- **Build:** `<command>`
- **Lint / format:** `<command>` (or "n/a")

## Live iterate (after every code change)

After any meaningful code change in the development phase:

1. Confirm dev server is live; check console — must be error-free.
2. Call `render_application_to_text()` and verify state matches the change.
3. For time-dependent changes, step with `advanceTime(ms)` and re-read.
4. If visual, take a screenshot; save under `output/iterate/`.
5. Smoke-check adjacent state for regressions.
6. Hand back to the user with a one-line verdict and one focused question.

A change is **not done** until this loop has run.

## Append vs spawn a new milestone

When new work surfaces:

- **Append** an AC to the current milestone if the work is in-scope refinement.
- **Spawn** a new milestone if the work is out of scope but related; use `Depends on:` to capture ordering.
- **Inline** trivial fixes (typos, one-liners) on the current milestone.

When in doubt, prefer spawning. Do not bloat milestones.

## Minimum-viable doc mode

If the user pushes back on documentation overhead, downgrade — do **not** skip:

- One-line milestone entry (title + AC) is acceptable.
- `docs/STATE.md` updates remain mandatory.
- `docs/product-plan.md` and `docs/tech.md` must exist.
- framework runtime / language / stack ADRs cannot be skipped.

## What to do if `make-app` isn't loaded

If the skill is missing or didn't trigger this session:

1. Stop. Do not start coding.
2. Read this file in full.
3. Read `docs/STATE.md`, then `docs/product-plan.md`, then `docs/tech.md`.
4. Identify the current phase and the open milestone.
5. Tell the user the skill isn't loaded and recommend they install it.
6. If proceeding without the skill, apply the rules above manually and update `docs/STATE.md` at the end of the session.

## Last regenerated

<YYYY-MM-DD by <agent or user>. Regenerate when the framework runtime, primary commands, or architecture rules change. The doc-drift audit will flag staleness.>
