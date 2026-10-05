---
name: scaffold-entitlements
description: Scaffold feature flags, plans, roles, quotas, and entitlements for a React web application without binding the UI to a billing provider. Use when the user asks for gated features, tiers, permissions, or premium hooks.
argument-hint: "[entitlement model]"
license: MIT
metadata:
  tags: [react, entitlements, feature-flags, permissions]
---

# Scaffold Entitlements

Keep entitlement evaluation explicit and server-authoritative.

- Define capabilities and their default behavior before adding gates.
- Expose a typed entitlement context or hook for presentation only; enforce access in the API as well.
- Provide allowed, unavailable, loading, expired, and unauthorized states without hiding core navigation.
- Keep plan/provider identifiers out of business components where an adapter can own them.
- Add tests for role boundaries, stale entitlement data, and direct API access.
- Make an upgrade or request-access path informative and accessible when appropriate.

## Full Workflow

1. Inventory roles, capabilities, quotas, plans, default access, trial/expiry rules, and server enforcement points.
2. Define a typed entitlement contract and explicit behavior for allowed, denied, loading, stale, expired, and unauthorized states.
3. Write tests for role boundaries, direct API access, stale data, feature flag changes, and quota edges.
4. Implement server evaluation first, then expose a small presentation hook/context to React components.
5. Add accessible upgrade, request-access, or contact-admin states without hiding core navigation unexpectedly.
6. Verify cache invalidation after billing or role changes and test an account with no entitlements.
7. Record the model and provider-neutral assumptions in `docs/tech.md` or an ADR.