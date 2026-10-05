---
name: manage-web-assets
description: Add, organize, optimize, and integrate images, fonts, video, icons, and documents in a React web application. Use when the user asks to add web assets or improve asset delivery.
argument-hint: "[asset type or task]"
license: MIT
metadata:
  tags: [react, assets, images, fonts, performance]
---

# Manage Web Assets

Make assets discoverable, licensed, responsive, and cheap to deliver.

- Inspect the current public/src asset convention and bundler behavior.
- Choose format, intrinsic dimensions, responsive variants, lazy loading, and cache policy.
- Avoid layout shift with dimensions or aspect ratios; provide alt text or mark decorative assets correctly.
- Subset fonts and preload only critical resources.
- Keep large or user-uploaded assets behind a server/CDN boundary with validation and size limits.
- Verify production URLs and network behavior after build.

## Full Workflow

1. Inventory the asset type, owner, license, dimensions, variants, route usage, and delivery requirements.
2. Inspect bundler, public/src conventions, CDN, upload endpoint, cache, and existing metadata handling.
3. Define validation and failure behavior before implementation; never trust client filenames or MIME types.
4. Add dimensions/aspect ratios, alt text or decorative semantics, lazy loading, and font preload only when justified.
5. Implement the smallest pipeline change and test missing, slow, invalid, and production-path assets.
6. Check bundle size, network waterfall, layout shift, responsive rendering, and cache headers.
7. Record asset decisions, source/license metadata, and the next cleanup or optimization task.