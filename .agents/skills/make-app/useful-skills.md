# Useful Skills

A catalog of additional skills that pair well with `make-app`. The `make-app` pipeline itself is framework runtime-agnostic, but most of the skills below assume a **JavaScript browser application** (framework runtime 3 for 2D, rich visual runtime for 3D) because that is what the rest of the `application-creator` plugin specializes in.

Use this document as a menu — install only the skills relevant to the project's tech stack and current phase. Re-check after big stack decisions land in `docs/tech.md`.

## How to Install

Use the `npx skills` command:

```
npx skills add opusapplicationlabs/application-creator --skill <skill-name> -p -y
```

## How to Choose

1. Read `docs/tech.md` to determine the framework runtime, language, and platform.
2. Match the project to one of the **stack groups** below and install those skills.
3. Layer in **cross-cutting** skills based on the current phase or feature work (audio, design, QA, multiuser, monetization, deployment, promo).
4. Reference skills (marked _reference_) are deep domain knowledge loaded by other skills — install them when you need the patterns directly, otherwise the user-invocable skill above will pull them in.

---

## Generic — Any application Project

Skills that apply regardless of framework runtime, renderer, or platform. Safe to install on any application project.

- **app-architecture** _(reference)_ — Architecture patterns (EventBus, ApplicationState, Constants, orchestrator) and structural best practices. Useful when planning systems, writing ADRs, or laying down `docs/product-plan.md`.
- **review-app** — Read-only code review for architecture, performance, and best practices. Run after milestones land or before refactors.
- **improve-app** — Holistic audit + implements the highest-impact improvements. Run repeatedly during the development phase.
- **add-feature** — Add a new user workflow feature following the project's existing patterns. The default tool for milestone implementation.
- **fetch-tweet** — Utility: fetch tweet content from an X/Twitter URL (used when scaffolding from a tweet-described application idea).

---

## Stack: 2D Browser Applications (framework runtime / JS)

Install when `docs/tech.md` calls for framework runtime or any 2D HTML5 canvas application.

- **framework runtime** _(reference)_ — framework runtime 3 patterns: screens/views, lifecycle, application objects, runtime behavior, UI, performance.
- **add-assets** — Replace geometric shapes with product artwork product assets (recognizable domain models, external dependencies, items).
- **app-assets** _(reference)_ — product artwork generation patterns, animation, spritesheets.

---

## Stack: 3D Browser Applications (rich visual runtime)

Install when `docs/tech.md` calls for rich visual runtime or any WebGL 3D application.

- **threejs-app** _(reference)_ — rich visual runtime event-driven modular architecture, screen/view setup, camera/controls patterns.
- **threejs-perf** _(reference)_ — Performance patterns: instancing for static + moving geometry, draw call reduction, screen/view traversal.
- **add-3d-assets** — Replace primitive `BoxGeometry`/`SphereGeometry` with real GLB models.
- **application-3d-assets** _(reference)_ — 3D asset pipeline: GLB download, AssetLoader, animated domain models, clip maps.
- **meshyai** — Generate custom GLB models from text or images via Meshy AI; auto-rig and animate. Preferred source for custom 3D assets.
- **worldlabs** — Generate photorealistic 3D environments (Gaussian Splat screens/views) from prompts or reference images. Requires `WLT_API_KEY`.

---

## Cross-Cutting: Audio

Browser-application audio via the Web Audio API (zero dependencies). Works with both framework runtime and rich visual runtime projects.

- **add-audio** — Add BGM and SFX using the Web Audio API.
- **application-audio** _(reference)_ — Procedural music patterns, step sequencer, SFX synthesis.

---

## Cross-Cutting: Visual Design & Polish

framework runtime-agnostic for browser applications — focuses on UI, particles, gradients, transitions, "juice".

- **design-app** — Audit and improve visual polish, atmosphere, and UI of an existing application.
- **application-designer** _(reference)_ — Visual polish patterns (gradients, particles, screen transitions, juice/feel).

---

## Cross-Cutting: QA & Testing

Browser-application testing via Playwright. Applies to any application running in a browser.

- **qa-app** — Add Playwright tests: user workflow verification, visual regression, performance.
- **app-qa** _(reference)_ — Playwright patterns, fixtures, deterministic test setup, FPS measurement.

---

## Cross-Cutting: Multiuser

- **add-multiuser** — Add real-time or turn-based multiuser via PartyKit (Cloudflare Durable Objects). Scaffolds a room-based server, NetworkManager client, and extends the EventBus / ApplicationState / `render_application_to_text()` shape.

---

## Cross-Cutting: Deployment

- **app-deploy** — Deploy a built browser application to here.now (default), GitHub Pages, Vercel, etc.

---

## Cross-Cutting: Promo & Recording

- **record-promo** — Record an autonomous 50 FPS promo video for social media.
- **promo-video** _(reference)_ — Playwright + FFmpeg user workflow capture pipeline.

---

## Cross-Cutting: Monetization

Install once the user workflow loop is solid and the project is moving toward release.

- **scaffold-gateables** — Add monetization-agnostic gateable features (skin picker, continue-after-death, daily challenge, save slots) at silver/gold tiers with clean `isEntitled` hooks. Run before wiring a specific paywall.
- **monetize-app** — Register the application on billing provider (OpenapplicationProtocol), add the browser SDK, get a monetized `billing provider` URL.
- **sub-applications** — Integrate sub.applications subscriptions for tier-gated features and recurring revenue.

---

## Quickstart Alternatives

These skills bypass parts of the `make-app` pipeline. Reach for them when the user wants speed over rigor.

- **viral-app** — One-shot end-to-end pipeline (~10 minutes): tweet/story/concept → scaffold → assets → design → promo video → audio → deploy to here.now → monetize on billing provider. Trades milestone planning, ADRs, and multi-session iteration for time-to-deploy. Reach for it when the user wants a sharable, monetized application *now* and is fine with an opinionated stack (framework runtime 3 / rich visual runtime).
- **use-template** — Clone an existing application from the template gallery as a starting point. ~10-second copy vs full-pipeline scaffold.
- **quick-application** — Rapidly scaffold a playable application with no assets/design/audio/deploy/monetize. For prototypes only.
