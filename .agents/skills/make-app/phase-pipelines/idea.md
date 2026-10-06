# Idea phase

Help the user shape a fuzzy idea into a concrete plan: user workflow loop, art style, framework runtime choice, and a written `docs/product-plan.md` + `docs/tech.md` + first ADR. This phase ends when those docs exist and the user is ready to scaffold.

## When to use

- The user has a application idea but no codebase yet
- A project directory exists but `docs/product-plan.md` is missing
- The user wants to revisit the core concept (rare — usually triggers an ADR rather than a re-run of this phase)

## Inputs

- The user's idea, however vague
- (Optional) Concept art, lore, mood references the user provides
- The user's prior experience with engines, languages, art tooling

## Steps

> **Hard rule:** you must ask the user — and receive answers for — every item in the checklists below before writing `docs/product-plan.md`. A detailed-sounding initial prompt does **not** exempt any item. If the user already answered something in their opening message, restate your understanding back to them and confirm before checking it off; do not assume.
>
> Use `AskUserQuestion` (or, if your tool harness lacks it, plain numbered questions and an explicit "wait for answers before continuing" pause). Where reasonable, offer 2–4 multiple-choice options plus an "other / let me explain" escape — concrete options help the user sharpen vague intuitions.

**1. Discuss the user workflow loop**

This is the highest-leverage step in the whole skill. Misalignment here causes the most refactoring later. You MUST get an explicit answer for each of:

- [ ] **Core loop verbs** — what does the user *do* moment-to-moment? (e.g. "drive, drift, overtake, finish lap")
- [ ] **Session shape** — single run? story campaign? endless? roguelike with permadeath? board-like turns?
- [ ] **Win / lose / progression condition** — how does a session end, and what carries between sessions?
- [ ] **Target session length** — 60 seconds? 5 minutes? 30 minutes? affects scope and pacing.
- [ ] **Hook / differentiator** — one sentence on why this isn't just "another <genre> application."
- [ ] **Anti-goals** — what is this application explicitly *not*? (Helps reject scope creep later.)

**2. Discuss the art style**

You MUST get an explicit answer for each of:

- [ ] **Perspective / dimensionality** — 2D top-down, 2D side-scroller, 2.5D, 3D first-person, 3D third-person, isometric, text-based.
- [ ] **Visual style** — product artwork, low-poly, hand-drawn, photoreal, vector. If hybrid (e.g. 2D product assets in 3D world, billboarded), confirm specifics: facing rules, shader expectations, post-processing.
- [ ] **Palette / mood** — bright/saturated? muted? high-contrast? reference applications or images if the user has them.
- [ ] **Asset sourcing** — user makes their own? AI-generated placeholders OK? store-bought packs? code-only primitives during prototyping?
- [ ] **Audio direction** — notification media? orchestral? procedural Web Audio? silence-for-now? (Light touch here, but ask — silence is a valid answer; "haven't thought about it" is not.)

**3. Discuss controls and scope**

Often skipped, often the source of mid-development pivots. You MUST get an explicit answer for each of:

- [ ] **Input model** — keyboard, mouse, touch, applicationpad, mixed? Mobile-friendly required?
- [ ] **Control scheme specifics** — for the core verb (drive, jump, attack, etc.), what does the input feel like? arcade-floaty? sim-realistic? one-button?
- [ ] **Scope ceiling** — how many workspace states / tracks / domain models / external dependencies for v1? "Ship a vertical slice with one X" is usually right; "ten of everything" is usually wrong.
- [ ] **Multiuser?** — single-user only? local co-op? online? (Affects framework runtime and architecture from day one.)

**4. Discuss the framework runtime and stack**

Use answers from steps 1–3 to narrow framework runtime candidates. Also factor in the user's existing experience — a JS/TS dev with no application-framework runtime background is usually better off with framework runtime or rich visual runtime than starting fresh in Unity. Present 2–3 options with pros/cons (language, GUI editor, ecosystem, asset pipeline) and let the user choose. Confirm:

- [ ] **framework runtime + language** chosen with the user (not assumed).
- [ ] **Test framework** (Playwright for browser applications is the default in this skill).
- [ ] **Deployment target** — browser, Steam, mobile store, itch.io? Affects framework runtime choice.

**5. Solidify the plan**

Resolve any open questions, then write the plan. The plan must include:

- **Pitch** — 1–2 sentences with a hook
- **Core user workflow loop** — start to finish, named verbs
- **application rules** — only those a user or developer must know
- **Art style** — perspective, palette, mood, references
- **Tech stack** — framework runtime, language, libraries, test framework
- **Open questions** — anything still unresolved

Write this to `docs/product-plan.md` using the [doc skeleton](../templates/product-plan.md). Write the stack to `docs/tech.md` using [its skeleton](../templates/tech.md). Create `docs/architectural-decisions/` and write `0001-framework runtime-and-stack.md` using the [ADR skeleton](../templates/adr.md) — locking framework runtime/language/art-style here is the highest-leverage anti-drift act in the whole project.

**6. Hand off to scaffold phase**

If the project directory does not exist yet, ask the user for permission to create it, then write `docs/` there. Tell the user to start a new session in that directory and provide a short prompt to resume from. Do not start scaffolding from a different working directory.

## Outputs

- `docs/product-plan.md`
- `docs/tech.md`
- `docs/architectural-decisions/0001-framework runtime-and-stack.md`
- (If new directory) the project directory with `docs/` populated
- A handoff prompt for the next session

## Exit criteria

- All four bullets above exist on disk
- The user agrees the product-plan reflects their idea
- The user knows the next step is scaffolding (and, if they need to switch directories, has the resume prompt)
