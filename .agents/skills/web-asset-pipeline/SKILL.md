---
name: web-asset-pipeline
description: Establish a scalable image, font, video, icon, and document asset pipeline for a React web application. Use when asset volume, upload flows, CDN delivery, or media performance needs a deliberate design.
argument-hint: "[pipeline concern]"
license: MIT
metadata:
  tags: [react, assets, cdn, uploads, performance]
---

# Web Asset Pipeline

Separate source assets, transformed variants, and delivery URLs.

- Validate uploads by type, size, dimensions, and content; never trust a filename or client MIME alone.
- Generate responsive variants and metadata at a controlled boundary.
- Use immutable cache keys, CDN transforms when appropriate, and lazy loading for below-the-fold media.
- Prevent layout shift with dimensions and provide alt text, captions, or transcripts.
- Track failed transforms and missing assets with observable errors.
- Test a representative upload, transformation failure, and production URL.

## Full Workflow

1. Map source files, upload boundary, transformation worker, metadata store, CDN, and UI consumer.
2. Define contracts for type, size, dimensions, content safety, variants, immutable keys, and deletion.
3. Write tests for valid input, rejected input, transform failure, retry/idempotency, missing asset, and access control.
4. Implement server-side validation and transformation before adding React upload/progress states.
5. Add cancellation, retry, progress, success, error, and empty states without leaking internal storage details.
6. Measure upload time, transform time, cache hit, layout stability, and production URL correctness.
7. Document operational limits, observability, retention, and the handoff state.