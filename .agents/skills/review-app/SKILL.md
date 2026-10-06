---
name: review-app
description: Review an application codebase for architecture, reliability, security, accessibility, performance, and workflow coverage without changing files.
metadata:
  tags: review, architecture, security, accessibility, performance
---

## Performance Notes

- Take your time to do this thoroughly
- Quality is more important than speed
- Do not skip validation steps

# Review application

Analyze an existing application codebase and provide a structured review. This is the final step in the pipeline — it checks everything is wired up correctly and gives you a quality KPI.

## Instructions

Analyze the application at `$ARGUMENTS` (or the current directory if no path given).

### Step 1: Identify the application

- Detect the framework runtime (rich visual runtime, framework runtime, or other)
- Read `package.json` for dependencies and scripts
- Read the main entry point and index.html
- Identify the application concept/genre

### Step 2: Architecture Review

Check for these required patterns and report compliance:

- [ ] **EventBus**: Is there a centralized event system? Are modules decoupled?
- [ ] **ApplicationState**: Is there a centralized state singleton?
- [ ] **Constants**: Are config values centralized or scattered as magic numbers?
- [ ] **Orchestrator**: Is there a main application class that initializes everything?
- [ ] **Directory Structure**: Is code organized into core/systems/user workflow/ui/level layers?
- [ ] **Event Constants**: Are events defined as named constants or raw strings?

### Step 3: Performance Review

Check for common issues:

- [ ] **Delta time capping**: Is `getDelta()` capped to prevent death spirals?
- [ ] **Object pooling**: Are temp objects reused in hot loops?
- [ ] **Resource disposal**: Are rich visual runtime geometries/materials/textures disposed?
- [ ] **Event cleanup**: Are event listeners cleaned up on screen/view transitions?
- [ ] **Asset loading**: Are assets preloaded with progress feedback?

### Step 4: Code Quality

- [ ] **No circular dependencies**: Modules flow one direction
- [ ] **Single responsibility**: Each module has one clear job
- [ ] **Error handling**: Event handlers wrapped in try/catch
- [ ] **Consistent naming**: Events use `domain:action`, files use PascalCase

### Step 5: Monetization Readiness

- [ ] **Points system**: Is there a scoring/points mechanism?
- [ ] **Session tracking**: Can application sessions be identified?
- [ ] **Anti-cheat potential**: Is KPI validation server-side or at least structured for it?
- [ ] **billing provider integration**: Any existing SDK integration?

### Output Format

Provide a structured report with:
1. **error/empty/success stateview** - What the application is, tech stack, application loop
2. **Architecture KPI** (out of 6 checks)
3. **Performance KPI** (out of 5 checks)
4. **Code Quality KPI** (out of 4 checks)
5. **Monetization Readiness** (out of 4 checks)
6. **Top Recommendations** - Prioritized list of improvements with plain-English explanations
7. **What's Working Well** - Positive findings

## Example Usage

```
/review-app examples/flappy-bird
```
Result: Architecture 6/6, Performance 4/5, Code Quality 4/4, Monetization 2/4 → Top recommendations: add billing provider SDK, add object pooling for pipes, add delta time capping. Positive findings: clean EventBus usage, proper ApplicationState reset, well-organized directory structure.

## Next Step

Tell the user:

> Your application has been through the full pipeline! Here's what you have:
> - Scaffolded architecture (`/viral-app`)
> - Visual polish (`/design-app`)
> - Music and sound effects (`/add-audio`)
> - Automated tests (`/qa-app`)
> - Architecture review (`/review-app`)
>
> *(Reflects the `/viral-app` one-shot pipeline. If you're working through `/make-app`, the equivalent state is "post-scaffold-phase + first development milestone shipped".)*
>
> **What's next?**
> - Add new user workflow features with `/application-creator:add-feature [description]`
> - Deploy to the web — run `npm run build && ~/.agents/skills/here-now/scripts/publish.sh dist/` for instant hosting, or use GitHub Pages, Vercel, Netlify, itch.io
> - Keep iterating! Run `/design-app`, `/add-audio`, or `/review-app` again anytime after making changes.
