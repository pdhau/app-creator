# UI State Machine

Use for complex async, multi-step, or permission-sensitive workflows.

1. List states, events, guards, side effects, and terminal outcomes.
2. Include loading, success, empty, error, retry, unauthorized, offline, and cancellation states.
3. Make impossible transitions unrepresentable where practical with TypeScript unions.
4. Keep server state separate from transient UI state.
5. Write a transition table and tests for happy path, retries, races, and unmount cleanup.
6. Render each meaningful state intentionally.