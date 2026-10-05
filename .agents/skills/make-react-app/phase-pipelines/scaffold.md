# Scaffold Phase Pipeline

Use this phase after the product and stack are agreed but before feature development.

1. Read `docs/product-plan.md`, `docs/tech.md`, and ADR 0001.
2. Use the official current scaffolder for the chosen React stack; do not hand-write package or bundler setup when an official command exists.
3. Install dependencies and preserve package-manager and script conventions.
4. Create the smallest app shell: providers, route, error boundary, styling tokens, and environment boundary.
5. Add `AGENTS.md` and `CLAUDE.md` with `sub-pipelines/agents-bootstrap.md`.
6. Run the dev server, open the browser, check console output, exercise the initial route, and verify a production build plus preview.
7. Record the result in `docs/STATE.md` and create the first milestone.

Completion: the app boots, initial UI renders, console is clean, build/preview pass, and one documented command runs it.