# Useful Skills

A catalog of additional skills that pair well with `make-app`. The pipeline is runtime-agnostic, but the default browser stack is **React with Vite or Next.js**. Use `threejs-app` only when the product explicitly needs a 3D surface.

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

## Stack: React Browser Applications (Vite / Next.js)

Install when `docs/tech.md` calls for React, Vite, Next.js, or a standard browser application.

- **vercel-react-best-practices** _(reference)_ — React and Next.js performance, data fetching, bundle, and rendering patterns.
- **vercel-composition-patterns** _(reference)_ — scalable component APIs, compound components, context, and render props.
- **vercel-react-view-transitions** _(reference)_ — route, screen, list, and shared-element transitions when they clarify continuity.
- **web-design-guidelines** _(reference)_ — accessibility, responsive UI, interaction, and visual quality review.
- **app-assets** _(reference)_ — product asset generation patterns, responsive variants, and accessible fallbacks.

---

## Stack: 3D Browser Applications (rich visual runtime)

Install when `docs/tech.md` calls for rich visual runtime or any WebGL 3D application.

- **threejs-app** _(reference)_ — rich visual runtime event-driven modular architecture, screen/view setup, camera/controls patterns.
- **threejs-perf** _(reference)_ — Performance patterns: instancing for static + moving geometry, draw call reduction, screen/view traversal.
- **add-3d-assets** — Replace primitive `BoxGeometry`/`SphereGeometry` with real GLB models.
- **app-3d-assets** _(reference)_ — 3D asset pipeline: GLB download, AssetLoader, animated domain models, clip maps.
- **meshyai** — Generate custom GLB models from text or images via Meshy AI; auto-rig and animate. Preferred source for custom 3D assets.
- **worldlabs** — Generate photorealistic 3D environments (Gaussian Splat screens/views) from prompts or reference images. Requires `WLT_API_KEY`.

---

## Cross-Cutting: Audio

Browser-application audio via the Web Audio API (zero dependencies). Works with React, Next.js, and rich visual runtime projects.

- **add-audio** — Add BGM and SFX using the Web Audio API.
- **app-audio** _(reference)_ — Notification, transactional media, and user-controlled sound patterns.

---

## Cross-Cutting: Visual Design & Polish

Runtime-agnostic for browser applications — focuses on information hierarchy, feedback states, responsive behavior, and transitions.

- **design-app** — Audit and improve visual polish, atmosphere, and UI of an existing application.
- **app-designer** _(reference)_ — Product experience patterns for hierarchy, feedback, accessibility, and transitions.

---

## Cross-Cutting: QA & Testing

Browser-application testing via Playwright. Applies to any application running in a browser.

- **qa-app** — Add Playwright tests: user workflow verification, visual regression, performance.
- **app-qa** _(reference)_ — Playwright patterns, fixtures, deterministic test setup, FPS measurement.

---

## Cross-Cutting: Multiuser

- **add-multiplayer** — Add real-time or turn-based collaboration via PartyKit. For React apps, expose collaboration state through a provider/store and test two browser clients.

---

## Cross-Cutting: Deployment

- **app-deploy** — Deploy a built browser application to here.now, GitHub Pages, Vercel, or another target.
- **deploy-to-vercel** — Deploy an application to Vercel and return a preview or production URL.
- **vercel-cli-with-tokens** — Use token-based Vercel CLI authentication in automation.
- **vercel-optimize** — Investigate Vercel cost and performance issues from verified metrics.

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

- **viral-app** — One-shot end-to-end pipeline (~10 minutes): tweet/story/concept → React scaffold → assets → design → demo → notifications/media → deploy → monetize. Trades milestone planning, ADRs, and multi-session iteration for time-to-deploy.
- **use-template** — Clone an existing application from the template gallery as a starting point. ~10-second copy vs full-pipeline scaffold.
- **quick-app** — Rapidly scaffold a small React application with one useful workflow. For prototypes only.
