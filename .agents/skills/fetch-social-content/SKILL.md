---
name: fetch-social-content
description: Fetch and normalize public social content for a React web application or content workflow. Use when the user provides a post URL or asks to import social text, media, author, or engagement data.
argument-hint: "[public URL]"
license: MIT
metadata:
  tags: [web, social, content, import]
---

# Fetch Social Content

Use an approved API or server-side adapter rather than scraping from the browser.

- Validate the URL and provider before fetching.
- Normalize author, text, media, timestamp, and engagement fields into a typed model.
- Handle deleted, private, rate-limited, malformed, and unavailable content.
- Keep credentials server-side and respect provider terms, privacy, and content rights.
- Cache only what the product needs and show the source and freshness to users.
- Add a fixture-based test for the import parser.

## Full Workflow

1. Confirm provider, URL shape, legal/terms constraints, required fields, and whether the import runs server-side.
2. Define a typed normalized model and explicit behavior for private, deleted, rate-limited, malformed, and unavailable posts.
3. Add provider fixtures and failing parser/normalizer tests before implementation.
4. Implement URL validation, provider adapter, normalization, cache policy, and redaction at the server boundary.
5. Connect the React UI with loading, empty, error, retry, freshness, and source attribution states.
6. Verify credentials never reach the browser and test malformed provider responses and quota failures.
7. Document provider assumptions, content rights, and the next integration step.