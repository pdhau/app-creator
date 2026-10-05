---
name: web-deploy
description: Deploy a React web application to its configured hosting platform. Use when the user asks to publish, host, preview, or configure a production web build.
argument-hint: "[platform or release]"
license: MIT
compatibility: Requires the platform CLI or dashboard access and internet access.
metadata:
  tags: [react, deploy, hosting, vite, ci]
---

# Web Deployment

Validate the build before publishing.

- Inspect package scripts, framework base path, environment variables, redirects, headers, and adapter conventions.
- Run install, tests, typecheck/lint, `npm run build`, and a production preview when available.
- Confirm asset URLs, client-side route fallback, API origins, security headers, and source-map policy.
- Deploy through the repository's existing CI or platform adapter; do not expose secrets.
- Smoke-test the published URL on desktop and mobile and record rollback instructions.
- Never treat a local development server as production validation.

## Full Workflow

1. Read `docs/tech.md`, deployment scripts, CI configuration, environment contract, and target platform constraints.
2. Confirm build command, Node/package-manager version, base path, redirects, headers, API origins, and secrets.
3. Run focused tests, typecheck/lint, production build, and preview smoke test before deployment.
4. Check asset URLs, client-side route fallback, error reporting, source-map policy, and security headers.
5. Deploy through the existing CI or platform adapter and record the release identifier.
6. Smoke-test published routes on desktop/mobile, inspect console/network errors, and verify rollback instructions.
7. Update release notes, `docs/STATE.md`, known limitations, and next operational task.