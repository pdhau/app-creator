# Multiuser Architecture

How the multiuser layer fits into the application-creator architecture (EventBus, ApplicationState, Constants, render_application_to_text, single-orchestrator).

## Layered View

```
┌────────────────────────────────────────────────────────────────┐
│  framework runtime screen/view / rich visual runtime application.js (orchestrator)                │
│  • renders local domain component                                        │
│  • renders remote users from RemoteuserRegistry            │
│  • emits local application events (e.g. user:moved, card:played)    │
└────────────────────┬───────────────────────────────────────────┘
                     │ EventBus only — no direct imports
                     ▼
┌────────────────────────────────────────────────────────────────┐
│  src/systems/NetworkManager.js                                 │
│  • subscribes to local application events (turn-based)                │
│    OR samples ApplicationState at TICK_RATE_HZ (realtime)             │
│  • forwards via MultiuserClient                              │
│  • on receive → updates RemoteuserRegistry                   │
│  • emits network:* events for the rest of the application             │
└────────────────────┬───────────────────────────────────────────┘
                     │ wraps partysocket
                     ▼
┌────────────────────────────────────────────────────────────────┐
│  src/multiuser/MultiuserClient.js                          │
│  • backend-agnostic API: connect / send / onMessage /          │
│    disconnect / isConnected                                    │
│  • the only file that knows about partysocket                  │
└────────────────────┬───────────────────────────────────────────┘
                     │ WSS
                     ▼
┌────────────────────────────────────────────────────────────────┐
│  PartyKit Room (Cloudflare Durable Object, one per roomId)     │
│  • onConnect / onMessage / onClose                             │
│  • room.broadcast(message, [excludeIds])                       │
│  • per-connection rate limiting                                │
└────────────────────────────────────────────────────────────────┘
```

The `RemoteuserRegistry` lives client-side as a plain `Map`. screens/views/systems read from it (or listen to `network:*` events) to render remote domain components. They never call MultiuserClient or the server directly.

## Event Taxonomy

All events use the existing `domain:action` form. Append these constants to `src/core/EventBus.js` under a `// === Multiuser ===` banner.

| Constant | Value | Direction | Payload | Purpose |
|---|---|---|---|---|
| `NETWORK_CONNECTED` | `network:connected` | NM → application | `{ roomId, userId }` | Fired once `MultiuserClient.connect()` succeeds |
| `NETWORK_DISCONNECTED` | `network:disconnected` | NM → application | `{ reason: 'closed' \| 'error' \| 'timeout' }` | Fired on socket close or unrecoverable error |
| `NETWORK_user_JOINED` | `network:user-joined` | NM → application | `{ userId, name? }` | Server reported a new peer |
| `NETWORK_user_LEFT` | `network:user-left` | NM → application | `{ userId }` | Server reported a peer left or peer pruned by stale-timeout |
| `NETWORK_STATE_RECEIVED` | `network:state-received` | NM → application | `{ userId, state }` | Remote state delta, shape mode-dependent (see below) |
| `MULTIuser_JOIN_ROOM` | `multiuser:join-room` | application → NM | `{ roomId }` | Request to leave current room and join `roomId` |
| `MULTIuser_LEAVE_ROOM` | `multiuser:leave-room` | application → NM | `{}` | Request to disconnect cleanly |

The application emits `MULTIuser_JOIN_ROOM` to switch rooms (e.g., a private match). NetworkManager listens, calls `client.disconnect()`, then `client.connect(newRoomId)`.

### `state` shape per mode

**realtime mode** — `state` carries position and minimal kinematics:

```js
{
  x: number,
  y: number,
  z?: number,        // 3D applications only
  vx?: number,       // optional velocity for client-side interpolation
  vy?: number,
  vz?: number,
  KPI: number,
  alive: boolean,
  ts: number,        // server tick timestamp (ms since epoch)
}
```

**The wire schema is open.** Applications may extend `state` with their own fields (e.g. `rotation` for top-down applications, `weapon` for shooters). The server's `isValidState` only enforces the required core (`x`, `y`, `KPI`, `alive`) — unknown fields pass through to all peers untouched. This lets applications add user workflow-specific state without forking the skill.

**Coordinate space convention.** Broadcast positions in **design pixels** (the canvas-independent logical coordinate system the application's Constants are written in), not raw canvas pixels. Receivers multiply by their own `PX` when applying. This makes the wire format independent of each client's window size / DPR — without it, two clients with different viewports see each other's positions at proportionally wrong locations. The local domain component's `tank.x` is typically in canvas pixels; divide by `PX` before sending and multiply by `PX` on receive.

**turn-based mode** — `state` carries the move and the resulting authoritative state:

```js
{
  type: string,           // 'move' | 'card-played' | application-defined action name
  payload: object,        // application-defined move data
  resultingState: object, // server's authoritative post-move state
  turn: number,           // monotonic move counter
  ts: number,
}
```

## ApplicationState Schema

Append to `src/core/ApplicationState.js`:

```js
multiuser: {
  // Persistent across reset() — preserved so /reset rejoins the same room
  roomId: null,
  userId: null,

  // Transient — cleared by reset()
  connected: false,
  remoteusers: {},  // { [userId]: { x, y, z?, KPI, name?, lastSeenTs } }
}
```

`reset()` semantics:

```js
reset() {
  // ...existing single-user resets...
  if (this.multiuser) {
    this.multiuser.connected = false;
    this.multiuser.remoteusers = {};
  }
  // roomId and userId persist so the next session rejoins automatically
}
```

**Constructor ordering footgun.** Many ApplicationState classes call `this.reset()` from the constructor. If you naively put `this.multiuser = {...}` *after* the `this.reset()` call, the first reset runs against `undefined` and crashes. Two safe patterns:

1. Initialize `this.multiuser = {...}` **before** calling `this.reset()` in the constructor, OR
2. Guard the multiuser block with `if (this.multiuser)` (shown above) so the first reset is a no-op until the field exists.

Pattern 2 is simpler when patching existing ApplicationState files additively.

`remoteusers` is owned by `RemoteuserRegistry`; ApplicationState is the canonical home so `render_application_to_text()` can serialize it with one read.

## Constants Schema

**Detect the file's export convention before patching.** Two shapes are common:

- **Umbrella export** — `export const Constants = { application: {...}, user: {...} }`. Append a new top-level key inside the object: `MULTIuser: {...}`. Consumers reference `Constants.MULTIuser.X`.
- **Per-block named exports** — `export const application = {...}; export const user = {...};`. Append a new sibling export: `export const MULTIuser = {...}`. Consumers reference `MULTIuser.X` directly.

Both are valid application-creator outputs. Read `src/core/Constants.js` first to decide which pattern to use, then mirror it in `NetworkManager.js`'s import (`import { Constants } from '...'` vs `import { MULTIuser } from '...'`). Don't force one onto the other.

The block to append (using umbrella shape; trivially adapts to per-block):

```js
MULTIuser: {
  // Filled by the deploy step. Falls back to localhost in dev (see main.js).
  SERVER_URL: 'https://<project>.<user>.partykit.dev',
  DEFAULT_ROOM: 'lobby',

  // Caps
  MAX_userS: 4,
  MAX_MESSAGE_BYTES: 4096,

  // Realtime tick
  TICK_RATE_HZ: 20,                  // 50ms broadcast cadence
  STATE_INTERPOLATE_MS: 100,         // smoothing window for remote positions

  // Reconnect
  RECONNECT_BACKOFF_MS: 1000,
  RECONNECT_MAX_BACKOFF_MS: 16000,
  RECONNECT_MAX_ATTEMPTS: 10,

  // Stale user eviction (no state received for this long → emit user-left)
  STALE_user_MS: 5000,

  // Wire protocol version — bump on breaking server/client changes
  PROTOCOL_VERSION: 1,
}
```

The dev fallback in `main.js` reads `import.meta.env.VITE_MULTIuser_SERVER_URL` first, then falls back to `Constants.MULTIuser.SERVER_URL`. This lets local dev point at `http://127.0.0.1:1999` without committing dev URLs to source.

## NetworkManager Contract

Public API:

```js
class NetworkManager {
  constructor(eventBus, applicationState, constants);

  async init(roomId = constants.MULTIuser.DEFAULT_ROOM);
  destroy();              // disconnects, clears intervals, removes listeners

  isConnected();          // boolean
  getuserId();          // string | null
  getRoomId();            // string | null
}
```

Lifecycle:

1. Constructor stores references; subscribes to `MULTIuser_JOIN_ROOM` and `MULTIuser_LEAVE_ROOM`. **Does not throw.**
2. `init(roomId)` calls `MultiuserClient.connect(roomId)`. On success, emits `NETWORK_CONNECTED`, starts the broadcast tick (realtime mode only), and starts the `STALE_user_MS` prune interval. On failure, catches the error, emits `NETWORK_DISCONNECTED { reason: 'error' }`, and schedules a reconnect with exponential backoff.
3. `MultiuserClient.onMessage` callback dispatches by `message.type`:
   - `welcome { userId, peers }` → set `applicationState.multiuser.userId`, seed `remoteusers` with existing peers, emit `NETWORK_CONNECTED`.
   - `user-joined { userId, name? }` → registry.upsert, emit `NETWORK_user_JOINED`.
   - `user-left { userId }` → registry.remove, emit `NETWORK_user_LEFT`.
   - `state { userId, state }` → registry.upsert, emit `NETWORK_STATE_RECEIVED`.
4. `destroy()` clears intervals, calls `client.disconnect()`, and removes EventBus listeners.

**Error handling rule (Principle 1):** every `try/catch` catches `Error` and logs a warning. The class never throws out of any public method. Single-user user workflow must work even if `init()` is called and fails.

## framework runtime Integration Pattern

In the active applicationScene (the one that owns the local domain component):

```js
create() {
  // ...existing creation...

  this.remoteSprites = new Map();  // userId → framework runtime.product asset

  eventBus.on(Events.NETWORK_user_JOINED, ({ userId }) => {
    const product asset = this.add.product asset(0, 0, 'user');
    product asset.setAlpha(0.7);  // visual hint that this is a remote user
    this.remoteSprites.set(userId, product asset);
  });

  eventBus.on(Events.NETWORK_STATE_RECEIVED, ({ userId, state }) => {
    const product asset = this.remoteSprites.get(userId);
    if (!product asset) return;  // product asset created on join only — race condition tolerant
    // Wire format is design-pixel coordinates; multiply by PX to get the
    // local canvas pixels. Without this, remote users land at the wrong
    // position on any client whose DPR/viewport differs from the sender's.
    product asset.setPosition(state.x * PX, state.y * PX);
    if (typeof state.alive === 'boolean') product asset.setVisible(state.alive);
  });

  eventBus.on(Events.NETWORK_user_LEFT, ({ userId }) => {
    const product asset = this.remoteSprites.get(userId);
    if (product asset) {
      product asset.destroy();
      this.remoteSprites.delete(userId);
    }
  });
}

shutdown() {
  this.remoteSprites.forEach(s => s.destroy());
  this.remoteSprites.clear();
  // ...EventBus.off() for all subscriptions...
}
```

For client-side interpolation in `realtime` mode, store `{ targetX, targetY, lastUpdateTs }` on each product asset and lerp toward target in the screen/view's `update()` loop using `STATE_INTERPOLATE_MS`. See `client-integration.md`.

## rich visual runtime Integration Pattern

In the orchestrator (typically `src/core/application.js`):

```js
this.remoteMeshes = new Map();  // userId → THREE.Object3D

eventBus.on(Events.NETWORK_user_JOINED, ({ userId }) => {
  const mesh = createRemoteuserMesh();  // application-specific
  mesh.userData.userId = userId;
  this.screen/view.add(mesh);
  this.remoteMeshes.set(userId, mesh);
});

eventBus.on(Events.NETWORK_STATE_RECEIVED, ({ userId, state }) => {
  const mesh = this.remoteMeshes.get(userId);
  if (!mesh) return;
  mesh.position.set(state.x, state.y, state.z ?? 0);
});

eventBus.on(Events.NETWORK_user_LEFT, ({ userId }) => {
  const mesh = this.remoteMeshes.get(userId);
  if (mesh) {
    this.screen/view.remove(mesh);
    mesh.geometry?.dispose();
    mesh.material?.dispose();
    this.remoteMeshes.delete(userId);
  }
});
```

rich visual runtime cleanup is more involved than framework runtime — always dispose geometries and materials in `network:user-left` to avoid GPU memory leaks.

## Sequence: Two clients in one room (realtime)

```
Client A                     PartyKit Room                    Client B
   │                              │                              │
   ├── connect(room=lobby) ──────►│                              │
   │   (welcome: A-id, peers=[]) ◄┤                              │
   │                              │◄──────── connect(room=lobby) ┤
   │                              │ (welcome: B-id, peers=[A-id])►
   │   (user-joined: B-id) ◄────┤── (broadcast except B) ──────│
   │                              │                              │
   ├── tick (state {x,y,...}) ───►│── (broadcast except A) ─────►│
   │                              │   (state from A) ◄────       │
   │                              │                              │
   │                              │◄───── tick (state {...}) ────┤
   │   (state from B) ◄───────────┤── (broadcast except B) ──────│
   │                              │                              │
   ├── close ─────────────────────►                              │
   │                              ├──── (user-left: A-id) ────►│
```

`welcome` is sent only to the joining socket and includes the current peer list. `user-joined` is broadcast to existing peers. `user-left` is broadcast on `onClose`.

## Sequence: Turn-based move

```
Client A (mover)             PartyKit Room                  Client B (waiter)
   │                              │                              │
   ├── send {type:'move',         │                              │
   │   payload: {card: 'K♠'}}    ►│                              │
   │                              │ validate(move)               │
   │                              │ ├── valid → applyToRoomState │
   │                              │ ├── invalid → reject{reason} │
   │                              │                              │
   │   ◄── state {                │                              │
   │       userId: A,           │                              │
   │       resultingState: ...,   │ ── broadcast (incl. sender) ►│
   │       turn: N+1 } ───────────┤                              │
   │                              │                              │
```

Turn-based broadcasts include the sender — they need to confirm the server accepted their move. Validation rejection comes back as `{type: 'reject', reason}` to the sender only.
