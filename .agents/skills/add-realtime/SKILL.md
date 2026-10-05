---
name: add-realtime
description: Add realtime collaboration, presence, notifications, or live updates to a React web application using a small backend adapter. Use when the user asks for realtime updates, chat, presence, collaboration, WebSocket, SSE, or live data.
argument-hint: "[realtime workflow]"
license: MIT
metadata:
  tags: [react, realtime, websocket, collaboration]
---

# Add Realtime

Treat transport as an implementation detail behind a typed `RealtimeClient`.

- Define events and payloads before wiring UI.
- Separate server state, client cache, connection state, and transient UI state.
- Handle connecting, connected, reconnecting, offline, stale, and unauthorized states.
- Add bounded reconnect backoff, cleanup on unmount, deduplication, and optimistic rollback where appropriate.
- Keep secrets and authorization on the server; validate all incoming payloads.
- Test reconnect, duplicate events, slow networks, and an unavailable server.

## Full Workflow

1. Document the realtime outcome, participants, permissions, event ordering, conflict policy, and stale-data behavior.
2. Inspect the existing API/auth/query conventions and choose WebSocket, SSE, or the existing provider through an adapter.
3. Define typed events, validation schemas, connection states, retry limits, and server-authoritative mutations.
4. Write tests for initial sync, live update, duplicate event, reconnect, authorization failure, offline mode, and cleanup.
5. Implement the adapter, then connect it to a feature hook/store without making transport details part of presentational components.
6. Add optimistic updates only with rollback and conflict handling.
7. Verify two browser contexts, tab close/unmount cleanup, slow network behavior, and production configuration.
8. Record the protocol decision and operational assumptions in the active milestone and technical docs.