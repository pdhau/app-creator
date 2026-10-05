# Agents Bootstrap

1. Read `docs/product-plan.md`, `docs/tech.md`, ADRs, and active milestones.
2. Generate root `AGENTS.md` with commands, architecture rules, testing rules, and documentation contract.
3. Generate root `CLAUDE.md` containing only a pointer to `AGENTS.md`.
4. Keep architecture rules in `docs/tech.md` or ADRs; `AGENTS.md` is a reflection, not the source of truth.
5. Verify another agent can find the dev, test, build, and preview commands.
6. Regenerate after a top-level stack or architecture change.