---
name: add-multiplayer
description: Add realtime collaboration to an application with shared domain state, authorization, presence, conflict handling, reconnection, and multi-client tests.
metadata:
  tags: realtime, collaboration, synchronization, authorization, testing
---

# Add Multiuser (PartyKit / Cloudflare Durable Objects)

Add real-time or turn-based multiuser to an existing single-user browser application. This skill scaffolds:

1. A **PartyKit server** (one Durable Object per room) deployed to Cloudflare's edge.
2. A **client `NetworkManager`** wired through EventBus that mirrors the existing `playfun.js` external-service pattern.
3. **Additive edits** to `EventBus`, `ApplicationState`, `Constants`, and `render_application_to_text()` — single-user user workflow must remain identical when the server is unreachable.

The default state is "single-user works." If the WebSocket connection fails, NetworkManager swallows the error and the application runs locally as before. When connected, remote users appear via `network:user-joined` and synchronize via `network:state-received`.

## Reference Files

- `architecture.md` — event taxonomy, ApplicationState schema, NetworkManager contract, framework runtime vs rich visual runtime placement notes.
- `partykit-server.md` — server templates (`realtime.ts` and `turn-based.ts`), state shape, broadcast helpers, rate limiting.
- `client-integration.md` — `MultiuserClient`, `NetworkManager`, `RemoteuserRegistry` source, EventBus/ApplicationState/Constants append patterns, `render_application_to_text` extension.
- `deploy.md` — `npx partykit dev` and `npx partykit deploy` walkthrough, capturing the deployed URL, `.env` handling, and client redeploy.

## Core Principles

These are rules, not guidelines:

1. **Single-user must work offline.** With the server unreachable, the application must boot, play, and reset normally. NetworkManager catches all connection errors and emits `network:disconnected` instead of throwing.
2. **Additive edits only.** Append to `EventBus.js`, `ApplicationState.js`, `Constants.js`, `main.js`, and `render_application_to_text()` under a `// === Multiuser ===` banner. Never rename, remove, or change existing fields.
3. **EventBus is the only seam.** NetworkManager talks to the rest of the application through events — no direct imports from screens/views, systems, or domain components into NetworkManager (or vice versa).
4. **Server is authoritative, but tolerant.** The PartyKit room owns the canonical room state. Clients send intents; the server validates and broadcasts. In `realtime` mode validation is light (last-write-wins). In `turn-based` mode validation is strict (rejects out-of-turn moves).
5. **Backend-agnostic client API.** All `partysocket` calls go through `MultiuserClient`. If a future user wants Colyseus or fly.io+ws, only `MultiuserClient.js` changes — application code does not.
6. **Default room is `'lobby'`.** No matchmaking UI in v1. Users override by emitting `multiuser:join-room` with a custom room id.

## Prerequisites

- An existing framework runtime 3 or rich visual runtime application scaffolded with this plugin (has `src/core/EventBus.js`, `src/core/ApplicationState.js`, `src/core/Constants.js`, `src/main.js` with `window.render_application_to_text()`).
- Node.js 18+.
- A Cloudflare account for `npx partykit deploy` (the CLI walks the user through login on first deploy; free tier is sufficient for prototyping).

## Instructions

The user wants to add multiuser to the application at `$ARGUMENTS` (or the current directory if no path given). Optional `--mode=realtime` (default) or `--mode=turn-based` chooses the server template.

### Step 0: Locate and read the application

Parse `$ARGUMENTS` for the application path and `--mode` flag. If no path, use cwd. Verify it's a application by reading `package.json` and confirming framework runtime or rich visual runtime dependency.

Read these files in full before touching anything:

- `package.json` — framework runtime + scripts.
- `src/main.js` — orchestrator, `window.render_application_to_text()`, `window.advanceTime()`.
- `src/core/EventBus.js` — exact event names already in use.
- `src/core/ApplicationState.js` — current state shape and `reset()` semantics.
- `src/core/Constants.js` — config block conventions.
- `progress.md` if present — pipeline context.

Then tell the creator one sentence confirming what you saw:

> application is **`<framework runtime>`** with **`<N>`** events and a `<user|bird|ship>` domain component. I'll add a multiuser layer that broadcasts the local `<domain component>`'s state at `TICK_RATE_HZ` and renders remote users from server broadcasts. Single-user will continue to work when the server is offline.

### Step 1: Choose sync mode

Pick the server template:

| Mode | When to use | Wire model |
|---|---|---|
| `realtime` (default) | Action applications, runners, dodgers, platformers, anything with continuous movement | Local `setInterval` at `TICK_RATE_HZ` broadcasts the local domain component's `{x, y, [z], KPI, state}`; server fans out; clients render last-known remote state |
| `turn-based` | Card applications, board applications, puzzles, anything with discrete moves | EventBus events (`user:moved`, `user:played-card`) forward as `{type, payload}` messages; server validates and broadcasts; clients apply on `network:state-received` |

If the user did not pass `--mode`, infer from the application's existing events. If you see continuous-position events (`bird:flap`, `user:moved`, position-updating runtime behavior), use `realtime`. If you see discrete actions (`card:played`, `move:submitted`), use `turn-based`. State the choice and proceed.

### Step 2: Scaffold the server

Create a sibling `multiuser-server/` directory inside the application project. See `partykit-server.md` for the full template content.

Create:

- `multiuser-server/partykit.json` — manifest with `name` (use the application's directory name), `main: "src/server.ts"`, `compatibilityDate`.
- `multiuser-server/package.json` — `partykit` dep, `dev`/`deploy` scripts.
- `multiuser-server/tsconfig.json` — minimal TypeScript config that PartyKit accepts.
- `multiuser-server/src/server.ts` — paste the appropriate template from `partykit-server.md` (`realtime` or `turn-based`).
- `multiuser-server/.gitignore` — `node_modules`, `.partykit`.

Run `cd multiuser-server && npm install` to install `partykit` (which provides `partysocket` for the client too via npm workspaces, but we'll add `partysocket` explicitly to the client).

### Step 3: Scaffold the client

Create three new files. See `client-integration.md` for the full source.

- `src/multiuser/MultiuserClient.js` — backend-agnostic interface around `partysocket` (`connect`, `send`, `onMessage`, `disconnect`, `isConnected`).
- `src/multiuser/RemoteuserRegistry.js` — `Map<userId, Remoteuser>` with `upsert`, `remove`, `prune(staleMs)`, `list()`.
- `src/systems/NetworkManager.js` — wires MultiuserClient ↔ EventBus, owns the broadcast tick (in `realtime` mode), handles reconnect with exponential backoff, emits `network:*` events.

Add `partysocket` to the application's `package.json` deps:

```bash
cd <application-path> && npm install partysocket
```

### Step 4: Append to existing core files

Make additive edits only. See `architecture.md` for full schemas and `client-integration.md` for the exact append blocks.

**`src/core/EventBus.js`** — append under `// === Multiuser ===` banner:

```js
// === Multiuser ===
NETWORK_CONNECTED: 'network:connected',
NETWORK_DISCONNECTED: 'network:disconnected',
NETWORK_user_JOINED: 'network:user-joined',
NETWORK_user_LEFT: 'network:user-left',
NETWORK_STATE_RECEIVED: 'network:state-received',
MULTIuser_JOIN_ROOM: 'multiuser:join-room',
MULTIuser_LEAVE_ROOM: 'multiuser:leave-room',
```

**`src/core/ApplicationState.js`** — append a `multiuser` field with persistent (`roomId`, `userId`) and transient (`connected`, `remoteusers`) parts. Update `reset()` to clear only the transient parts so rejoin works after a application restart.

**`src/core/Constants.js`** — append a `MULTIuser` block with `SERVER_URL` (filled by Step 6), `DEFAULT_ROOM`, `MAX_userS`, `TICK_RATE_HZ`, reconnect backoff, stale-user threshold, `PROTOCOL_VERSION`. No magic numbers — every value is a named constant.

**`src/main.js`** — instantiate NetworkManager after EventBus + ApplicationState, before the framework runtime starts. Expose `window.__NETWORK_MANAGER__` for tests. Extend `window.render_application_to_text()` to additively include `multiuser: {...}` and `remoteusers: [...]`.

### Step 5: Wire the local application into the network tick

Inspect existing events. The wiring depends on mode:

**`realtime`**: NetworkManager owns a `setInterval` at `TICK_RATE_HZ`. Each tick it reads the local domain component from `ApplicationState` and calls `client.send({type: 'state', payload: {...}})`. No EventBus subscription needed — it just samples ApplicationState. Add a single `network:state-received` listener in the relevant screen/view/system that calls `RemoteuserRegistry.upsert()` and triggers a re-render.

**`turn-based`**: NetworkManager subscribes to the application's existing move events (e.g., `card:played`, `move:submitted`) and forwards them. The screen/view/system listens for `network:state-received` and applies the validated remote move. Local optimistic UI is allowed, but the server is the source of truth.

In framework runtime applications, remote-user rendering happens in the active applicationScene — instantiate product assets on `network:user-joined`, update positions on `network:state-received`, destroy on `network:user-left`. In rich visual runtime applications, the active orchestrator (`application.js`) creates and updates remote-user meshes.

See `client-integration.md` for example screen/view patches for both engines.

### Step 6: Deploy the server

Run the dev server first to confirm everything works locally:

```bash
cd <application-path>/multiuser-server && npx partykit dev
```

This starts a local CF Worker emulator on `http://127.0.0.1:1999`. In another terminal, set `VITE_MULTIuser_SERVER_URL=http://127.0.0.1:1999` in `<application-path>/.env` and run the client (`cd <application-path> && npm run dev`).

For first-time deployment, the user must authenticate with PartyKit. **Always pass `--provider github`** — the default `clerk` flow is broken in 2026 (the dashboard.partykit.io callback was retired after Cloudflare absorbed PartyKit, and login hangs forever):

```bash
cd <application-path>/multiuser-server && npx partykit login --provider github
```

This uses GitHub's device-code OAuth flow. The CLI prints a code; the user visits `https://github.com/login/device`, pastes it, and authorizes. Credentials persist in `~/.partykit/config.json`. See `deploy.md` for the full walkthrough and troubleshooting.

After login, deploy:

```bash
cd <application-path>/multiuser-server && npx partykit deploy
```

Capture the deployed URL from the output (format: `https://<project>.<cloudflare-username>.partykit.dev`). The TLS cert may take 30-60 seconds to provision after the deploy reports success.

Update three places with the deployed URL:

1. `src/core/Constants.js` → `MULTIuser.SERVER_URL`
2. `<application-path>/.env` → `VITE_MULTIuser_SERVER_URL=https://...`
3. `<application-path>/.env.example` → `VITE_MULTIuser_SERVER_URL=https://your-project.your-username.partykit.dev`

Add `.env` to `.gitignore` if not already present.

See `deploy.md` for the full walkthrough including offline-first authentication and troubleshooting.

### Step 7: Redeploy the client

Reuse the existing host detection logic (same as `monetize-app` Step 5):

1. If `.herenow/state.json` exists → redeploy via `~/.agents/skills/here-now/scripts/publish.sh dist/`.
2. Else if `gh` is configured and the repo has a GitHub Pages workflow → `npx gh-pages -d dist`.
3. Else if `vercel` is configured → `vercel --prod`.
4. Else ask the user how they want to redeploy.

Always run `npm run build` first.

### Step 8: Verify

**Build cleanly:**

```bash
cd <application-path> && npm run build
cd multiuser-server && npm run build  # if a build script exists
```

**Single-user fallback (critical):** with the partykit dev server stopped, reload `http://localhost:3000`. The application must boot, play, and reset normally. Confirm `network:disconnected` fired and no uncaught errors in the console. **If the application depends on the server to start, you violated Principle 1 — revise.**

**Two-tab smoke test:** start `npx partykit dev` in one terminal and `npm run dev` in another. Open two browser tabs at `http://localhost:3000`. Confirm:

- Both tabs fire `network:connected` (check console).
- Each tab's `window.render_application_to_text()` includes the other tab in `remoteusers`.
- Moving the local domain component in tab A is reflected in tab B's remote-user rendering within `1000 / TICK_RATE_HZ * 2` ms.

**Reconnect:** kill the partykit dev server, wait, restart it. The client should reconnect within `RECONNECT_MAX_BACKOFF_MS` and re-emit `network:connected`.

**Regression:** existing `tests/e2e/*.spec.js` must still pass. Single-user invariants (boot, KPI, application-over, reset) must hold whether the server is up or down.

### Step 9: Update `progress.md`

Append a `## Multiuser` section:

```markdown
## Multiuser

- **Backend**: PartyKit (Cloudflare Durable Objects)
- **Server URL**: https://<project>.<user>.partykit.dev
- **Mode**: realtime | turn-based
- **Max users per room**: 4
- **Tick rate**: 20 Hz (realtime mode)
- **Default room**: lobby
- **Known limitations (v1)**: no matchmaking UI, no spectator mode, no persistent accounts, server-side rate limiting only.
```

## Output

Tell the user:

1. **What was added** — server in `multiuser-server/`, client in `src/multiuser/` + `src/systems/NetworkManager.js`, additive edits to four core files.
2. **The server URL** — `https://<project>.<user>.partykit.dev`. Already wired into Constants and `.env`.
3. **How to test locally** — `cd multiuser-server && npx partykit dev` then `npm run dev`, open two tabs.
4. **The single seam for backend swaps** — point at `src/multiuser/MultiuserClient.js`. Future Colyseus or fly.io migration only changes that one file.
5. **Costs** — free on Cloudflare's Workers free tier (100k requests/day, 1GB DO storage). Mention the user owns the deployed Cloudflare project; PartyKit deploys to their CF account, not OpusapplicationLabs'.

## Example Usage

### Default (realtime mode in current directory)

```
/add-multiuser
```

Result: detects framework runtime, scaffolds `multiuser-server/` with the realtime template, creates client networking files, deploys server, redeploys client, prints play URL and server URL.

### Turn-based explicit

```
/add-multiuser ./examples/card-application --mode=turn-based
```

Result: uses the turn-based server template; NetworkManager forwards the application's existing move events instead of running a position-broadcast tick.

### Verbose dry-run for inspection

```
/add-multiuser --dry-run
```

Result: prints the full file list and patches without writing or deploying. Useful for review before committing.

## Troubleshooting

### `npx partykit login` redirects to `dashboard.partykit.io/patience` and never completes
**Cause:** The default `clerk` provider was retired after Cloudflare absorbed PartyKit; the dashboard the OAuth callback expects is gone.
**Fix:** Use `npx partykit login --provider github` instead — GitHub device-code flow, prints a code, you paste at `https://github.com/login/device`. Credentials persist in `~/.partykit/config.json`.

### Remote users don't appear even though the connection succeeded
**Cause:** Welcome-race — the WebSocket `welcome` arrived before the screen/view's `create()` registered its `NETWORK_user_JOINED` listener. The events fired into the void.
**Fix:** After registering the listener, seed from `applicationState.multiuser.remoteusers` directly. See `client-integration.md` → "Welcome-race gotcha" for the idempotent pattern.

### Two tabs connect but never see each other
**Cause:** They joined different rooms (random room IDs from URL parsing) or the server's broadcast logic excludes the sender by default.
**Fix:** Check the room id in `window.render_application_to_text().multiuser.roomId` on both tabs — it should be the same (default `'lobby'`). If different, audit `NetworkManager.connect()` for stray query-string parsing. The server template's `room.broadcast(message, [sender.id])` excludes the sender, which is correct — each client renders only *remote* users, not itself.

### Remote users appear stuck at last position when a peer closes the tab
**Cause:** `onClose` did not fire (browser killed the tab without a clean close), or the client did not run `RemoteuserRegistry.prune()`.
**Fix:** The server's `onClose` handler is the canonical "user left" signal. Additionally, NetworkManager runs `RemoteuserRegistry.prune(STALE_user_MS)` on every tick — verify this is wired. If a remote user has not sent state in `STALE_user_MS`, prune emits `network:user-left` even without an explicit close.

### application lags or stutters when many remote users are present
**Cause:** Either too-high `TICK_RATE_HZ` (you're broadcasting and rendering 60 times per second) or the screen/view re-creates remote-user product assets every frame instead of reusing them.
**Fix:** Lower `TICK_RATE_HZ` to 20 (default) or 10 for slow applications. Confirm screens/views maintain a `Map<userId, product asset>` and only update positions on `network:state-received`, never recreate.

### Single-user tests fail after adding multiuser
**Cause:** NetworkManager throws or blocks application boot when the server is unreachable. This violates Principle 1.
**Fix:** Audit `MultiuserClient.connect()` and `NetworkManager.init()` — both must catch all errors, log a warning, emit `network:disconnected`, and return. The constructor and `init()` must never throw out of `main.js`.

### `render_application_to_text()` snapshot tests fail
**Cause:** Tests use exact `toEqual` on the output; you added new top-level fields.
**Fix:** Regenerate baselines — additions are intentional and backward-compatible. The fields added are `multiuser` (object) and `remoteusers` (array, may be empty).

### Cloudflare deploy succeeds but the WebSocket fails in the browser
**Cause:** Mixed content (HTTP page → WSS server) or the server URL was written without the `https://` scheme.
**Fix:** Confirm `Constants.MULTIuser.SERVER_URL` is the full `https://...partykit.dev` URL. `partysocket` derives the WSS URL by replacing the scheme. The deployed application must also be served over HTTPS for the WSS connection to succeed (here.now and GitHub Pages both serve HTTPS by default).

### Free tier rate limit hit
**Cause:** Many concurrent rooms or chatty clients.
**Fix:** Cloudflare's Workers free tier allows 100k requests/day. Each client tick at 20 Hz is one request — that's `1.7M / day` for a single 24/7 user. For prototyping you'll never hit this; for production, lower `TICK_RATE_HZ` or upgrade to Workers Paid ($5/mo flat for 10M requests/day).

## Tips

> Run `/add-multiuser` once per application. If you later change modes, edit `multiuser-server/src/server.ts` directly — both templates are checked in and the switch is small.
>
> The default `'lobby'` room is suitable for a single open room. To support private rooms, emit `multiuser:join-room` with a room id from a URL query string or invite code. NetworkManager listens for that event and reconnects to the new room.
>
> For graduation to a more featureful backend (matchmaking, schema sync, server-authoritative runtime behavior), the only file that needs to change is `src/multiuser/MultiuserClient.js`. Replace the `partysocket` calls with Colyseus's `colyseus.js` client; keep the same public API (`connect`, `send`, `onMessage`, `disconnect`, `isConnected`).
>
> The server runs in *your* Cloudflare account, not OpusapplicationLabs'. Costs and quotas accrue to you. PartyKit itself is open source and free; you only pay Cloudflare's pass-through pricing (free tier is generous).
