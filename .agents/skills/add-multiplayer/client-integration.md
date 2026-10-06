# Client Integration

Source code for the three new client-side files plus the additive patches to existing core files.

## File: `src/multiuser/MultiuserClient.js`

The single seam between application code and the wire protocol. If a future user wants Colyseus or fly.io+ws, only this file changes.

```js
// src/multiuser/MultiuserClient.js
// Backend-agnostic WebSocket client. Wraps partysocket.
// Public API: connect, send, onMessage, onOpen, onClose, disconnect, isConnected.
// All errors are caught and routed through onClose/onError callbacks — never thrown.

import PartySocket from 'partysocket';

export class MultiuserClient {
  constructor() {
    this.socket = null;
    this.handlers = {
      message: () => {},
      open: () => {},
      close: () => {},
      error: () => {},
    };
  }

  connect({ host, room }) {
    if (this.socket) this.disconnect();
    try {
      this.socket = new PartySocket({ host, room });
      this.socket.addEventListener('open', () => this.handlers.open());
      this.socket.addEventListener('close', (e) => this.handlers.close(e));
      this.socket.addEventListener('error', (e) => this.handlers.error(e));
      this.socket.addEventListener('message', (e) => {
        let parsed = null;
        try { parsed = JSON.parse(e.data); } catch { return; }
        this.handlers.message(parsed);
      });
    } catch (err) {
      console.warn('[MultiuserClient] connect failed', err);
      this.handlers.error(err);
    }
  }

  send(message) {
    if (!this.isConnected()) return false;
    try {
      this.socket.send(JSON.stringify(message));
      return true;
    } catch (err) {
      console.warn('[MultiuserClient] send failed', err);
      return false;
    }
  }

  onMessage(cb) { this.handlers.message = cb; }
  onOpen(cb) { this.handlers.open = cb; }
  onClose(cb) { this.handlers.close = cb; }
  onError(cb) { this.handlers.error = cb; }

  isConnected() {
    return this.socket?.readyState === 1;  // WebSocket.OPEN
  }

  disconnect() {
    if (!this.socket) return;
    try { this.socket.close(); } catch { /* ignore */ }
    this.socket = null;
  }
}
```

`partysocket` already implements automatic reconnect — but we run our own reconnect logic in `NetworkManager` so we can emit clear events and apply our own backoff. To disable partysocket's reconnect and rely solely on ours, pass `startClosed: true` to the constructor and call `socket.reconnect()` manually; for v1 we let partysocket handle reconnect and just observe via `open`/`close` events.

## File: `src/multiuser/RemoteuserRegistry.js`

Owns the `Map<userId, Remoteuser>` that the rest of the application reads. Keeps `applicationState.multiuser.remoteusers` in sync (passed in by `NetworkManager`).

```js
// src/multiuser/RemoteuserRegistry.js
// Tracks remote users keyed by userId. Mirrors entries into applicationState.multiuser.remoteusers
// so render_application_to_text() can serialize with one read. Provides stale-eviction.

export class RemoteuserRegistry {
  constructor(applicationState) {
    this.applicationState = applicationState;
  }

  upsert(userId, partial) {
    const existing = this.applicationState.multiuser.remoteusers[userId] ?? {};
    this.applicationState.multiuser.remoteusers[userId] = {
      ...existing,
      ...partial,
      lastSeenTs: Date.now(),
    };
  }

  remove(userId) {
    delete this.applicationState.multiuser.remoteusers[userId];
  }

  has(userId) {
    return userId in this.applicationState.multiuser.remoteusers;
  }

  list() {
    return Object.entries(this.applicationState.multiuser.remoteusers).map(([id, p]) => ({ id, ...p }));
  }

  // Returns userIds pruned (so caller can emit user-left events).
  prune(staleMs) {
    const now = Date.now();
    const pruned = [];
    for (const [id, p] of Object.entries(this.applicationState.multiuser.remoteusers)) {
      if (now - (p.lastSeenTs ?? 0) > staleMs) {
        pruned.push(id);
        delete this.applicationState.multiuser.remoteusers[id];
      }
    }
    return pruned;
  }

  clear() {
    this.applicationState.multiuser.remoteusers = {};
  }
}
```

## File: `src/systems/NetworkManager.js`

Wires `MultiuserClient` ↔ `EventBus` ↔ `RemoteuserRegistry`. Owns the broadcast tick (realtime mode) and the prune interval. **Never throws out of any public method.**

```js
// src/systems/NetworkManager.js
// Single source of truth for network I/O. Subscribes to application events, forwards to server,
// dispatches server messages to EventBus + RemoteuserRegistry.
// Single-user must work when init() fails — every error is caught and surfaced via
// network:disconnected. user workflow never depends on the server being reachable.

import { Events } from '../core/EventBus.js';
// Adapt this import to match the application's Constants.js export shape:
//   - Umbrella:  import { Constants } from '../core/Constants.js'  →  Constants.MULTIuser.X
//   - Per-block: import { MULTIuser } from '../core/Constants.js' →  MULTIuser.X
// (Most application-creator scaffolds use per-block. Read Constants.js first.)
import { Constants } from '../core/Constants.js';
import { MultiuserClient } from '../multiuser/MultiuserClient.js';
import { RemoteuserRegistry } from '../multiuser/RemoteuserRegistry.js';

const MODE_REALTIME = 'realtime';
const MODE_TURN_BASED = 'turn-based';

export class NetworkManager {
  // mode: 'realtime' | 'turn-based'
  // sampler: () => object   — only used in realtime mode; returns the local user's state slice
  // moveEvents: string[]    — only used in turn-based mode; EventBus event names whose payloads to forward
  constructor({ eventBus, applicationState, mode, sampler, moveEvents = [] }) {
    this.eventBus = eventBus;
    this.applicationState = applicationState;
    this.mode = mode;
    this.sampler = sampler ?? (() => null);
    this.moveEvents = moveEvents;

    this.client = new MultiuserClient();
    this.registry = new RemoteuserRegistry(applicationState);

    this.tickInterval = null;
    this.pruneInterval = null;
    this.reconnectAttempts = 0;
    this.reconnectTimer = null;
    this.intentionalDisconnect = false;

    this.boundHandlers = {};
  }

  async init(roomId = Constants.MULTIuser.DEFAULT_ROOM) {
    this.applicationState.multiuser.roomId = roomId;
    this._wireClient();
    this._wireapplicationEvents();
    this._connect(roomId);
  }

  destroy() {
    this.intentionalDisconnect = true;
    this._stopTick();
    this._stopPrune();
    this._cancelReconnect();
    this._unwireapplicationEvents();
    this.client.disconnect();
    this.registry.clear();
    this.applicationState.multiuser.connected = false;
  }

  isConnected() {
    return this.client.isConnected();
  }

  getuserId() {
    return this.applicationState.multiuser.userId;
  }

  getRoomId() {
    return this.applicationState.multiuser.roomId;
  }

  // ---- private ----

  _wireClient() {
    this.client.onOpen(() => {
      this.reconnectAttempts = 0;
      this._cancelReconnect();
    });

    this.client.onMessage((msg) => {
      switch (msg?.type) {
        case 'welcome':       return this._onWelcome(msg);
        case 'user-joined': return this._onuserJoined(msg);
        case 'user-left':   return this._onuserLeft(msg);
        case 'state':         return this._onState(msg);
        case 'turn':          return this._onTurn(msg);
        case 'reject':        console.warn('[NetworkManager] server rejected', msg.reason); return;
        default:              return;
      }
    });

    this.client.onClose(() => this._onSocketClosed());
    this.client.onError(() => { /* close will follow */ });
  }

  _wireapplicationEvents() {
    this.boundHandlers.joinRoom = ({ roomId } = {}) => {
      if (!roomId || roomId === this.applicationState.multiuser.roomId) return;
      this.applicationState.multiuser.roomId = roomId;
      // Not connected → just connect, nothing to hand off.
      if (!this.client.isConnected()) {
        this._connect(roomId);
        return;
      }
      // Already connected — defer the new connect to _onSocketClosed via
      // pendingRoomId so we don't race the old socket's async close event.
      // Without this, a delayed close fires AFTER intentionalDisconnect is
      // already cleared, gets misclassified as an error, emits a spurious
      // network:disconnected, and schedules a duplicate reconnect.
      this.pendingRoomId = roomId;
      this.intentionalDisconnect = true;
      this.client.disconnect();  // _onSocketClosed performs the handoff
    };
    this.boundHandlers.leaveRoom = () => this.destroy();

    this.eventBus.on(Events.MULTIuser_JOIN_ROOM, this.boundHandlers.joinRoom);
    this.eventBus.on(Events.MULTIuser_LEAVE_ROOM, this.boundHandlers.leaveRoom);

    if (this.mode === MODE_TURN_BASED) {
      this.boundHandlers.move = (payload) => {
        if (!this.client.isConnected()) return;
        this.client.send({ type: 'move', payload });
      };
      for (const evt of this.moveEvents) {
        this.eventBus.on(evt, this.boundHandlers.move);
      }
    }
  }

  _unwireapplicationEvents() {
    this.eventBus.off(Events.MULTIuser_JOIN_ROOM, this.boundHandlers.joinRoom);
    this.eventBus.off(Events.MULTIuser_LEAVE_ROOM, this.boundHandlers.leaveRoom);
    if (this.mode === MODE_TURN_BASED) {
      for (const evt of this.moveEvents) {
        this.eventBus.off(evt, this.boundHandlers.move);
      }
    }
  }

  _connect(roomId) {
    // Guard `import.meta` for non-Vite contexts (some test runners, SSR, plain
    // Node). Without the typeof check, this throws a SyntaxError at parse time
    // in environments where import.meta isn't a thing.
    const envHost = (typeof import.meta !== 'undefined' && import.meta.env?.VITE_MULTIuser_SERVER_URL) || null;
    const host = envHost ?? Constants.MULTIuser.SERVER_URL;
    if (!host) {
      console.warn('[NetworkManager] no SERVER_URL configured — multiuser disabled');
      this.applicationState.multiuser.connected = false;
      this.eventBus.emit(Events.NETWORK_DISCONNECTED, { reason: 'error' });
      return;
    }
    try {
      this.client.connect({ host, room: roomId });
    } catch (err) {
      console.warn('[NetworkManager] connect threw', err);
      this._scheduleReconnect();
    }
  }

  _onSocketClosed() {
    this._stopTick();
    this._stopPrune();
    this.applicationState.multiuser.connected = false;
    this.registry.clear();
    this.eventBus.emit(Events.NETWORK_DISCONNECTED, { reason: this.intentionalDisconnect ? 'closed' : 'error' });
    if (!this.intentionalDisconnect) this._scheduleReconnect();
    this.intentionalDisconnect = false;
  }

  _scheduleReconnect() {
    if (this.reconnectTimer) return;
    if (this.reconnectAttempts >= Constants.MULTIuser.RECONNECT_MAX_ATTEMPTS) {
      console.warn('[NetworkManager] reconnect attempts exhausted');
      return;
    }
    const base = Constants.MULTIuser.RECONNECT_BACKOFF_MS;
    const max = Constants.MULTIuser.RECONNECT_MAX_BACKOFF_MS;
    const delay = Math.min(base * Math.pow(2, this.reconnectAttempts), max);
    this.reconnectAttempts += 1;
    this.reconnectTimer = setTimeout(() => {
      this.reconnectTimer = null;
      this._connect(this.applicationState.multiuser.roomId ?? Constants.MULTIuser.DEFAULT_ROOM);
    }, delay);
  }

  _cancelReconnect() {
    if (this.reconnectTimer) {
      clearTimeout(this.reconnectTimer);
      this.reconnectTimer = null;
    }
  }

  _onWelcome({ userId, peers = [] }) {
    this.applicationState.multiuser.userId = userId;
    this.applicationState.multiuser.connected = true;
    for (const peer of peers) {
      this.registry.upsert(peer.userId, { name: peer.name, ...(peer.state ?? {}) });
      this.eventBus.emit(Events.NETWORK_user_JOINED, { userId: peer.userId, name: peer.name });
    }
    this.eventBus.emit(Events.NETWORK_CONNECTED, {
      roomId: this.applicationState.multiuser.roomId,
      userId,
    });
    if (this.mode === MODE_REALTIME) this._startTick();
    this._startPrune();
  }

  _onuserJoined({ userId, name }) {
    this.registry.upsert(userId, { name });
    this.eventBus.emit(Events.NETWORK_user_JOINED, { userId, name });
  }

  _onuserLeft({ userId }) {
    this.registry.remove(userId);
    this.eventBus.emit(Events.NETWORK_user_LEFT, { userId });
  }

  _onState({ userId, state }) {
    if (userId === this.applicationState.multiuser.userId) return;  // never render self
    this.registry.upsert(userId, state);
    this.eventBus.emit(Events.NETWORK_STATE_RECEIVED, { userId, state });
  }

  _onTurn({ userId, resultingState, turn }) {
    this.eventBus.emit(Events.NETWORK_STATE_RECEIVED, {
      userId,
      state: { type: 'turn', resultingState, turn, ts: Date.now() },
    });
  }

  _startTick() {
    if (this.tickInterval) return;
    const intervalMs = 1000 / Constants.MULTIuser.TICK_RATE_HZ;
    this.tickInterval = setInterval(() => {
      const sample = this.sampler();
      if (!sample) return;
      this.client.send({ type: 'state', state: { ...sample, ts: Date.now() } });
    }, intervalMs);
  }

  _stopTick() {
    if (this.tickInterval) {
      clearInterval(this.tickInterval);
      this.tickInterval = null;
    }
  }

  _startPrune() {
    if (this.pruneInterval) return;
    const intervalMs = 1000 / Constants.MULTIuser.TICK_RATE_HZ;
    this.pruneInterval = setInterval(() => {
      const pruned = this.registry.prune(Constants.MULTIuser.STALE_user_MS);
      for (const id of pruned) this.eventBus.emit(Events.NETWORK_user_LEFT, { userId: id });
    }, intervalMs);
  }

  _stopPrune() {
    if (this.pruneInterval) {
      clearInterval(this.pruneInterval);
      this.pruneInterval = null;
    }
  }
}
```

## EventBus.js — Append Block

Append to `src/core/EventBus.js` inside the `Events` constant, just before the closing `}`:

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

## ApplicationState.js — Append Block

Append to the ApplicationState constructor / initial state object:

```js
// === Multiuser ===
this.multiuser = {
  // Persistent across reset() — preserved so /reset rejoins the same room.
  roomId: null,
  userId: null,

  // Transient — cleared by reset().
  connected: false,
  remoteusers: {},  // { [userId]: { x, y, z?, KPI, name?, lastSeenTs } }
};
```

In `reset()`, append (with a `this.multiuser` guard — `reset()` is often called from the ApplicationState constructor, possibly *before* the `this.multiuser = {...}` block runs):

```js
// === Multiuser (transient only) ===
if (this.multiuser) {
  this.multiuser.connected = false;
  this.multiuser.remoteusers = {};
}
// roomId and userId persist intentionally.
```

## Constants.js — Append Block

Append to the `Constants` object:

```js
// === Multiuser ===
MULTIuser: {
  SERVER_URL: 'https://<project>.<user>.partykit.dev',  // filled by deploy step
  DEFAULT_ROOM: 'lobby',

  MAX_userS: 4,
  MAX_MESSAGE_BYTES: 4096,

  TICK_RATE_HZ: 20,
  STATE_INTERPOLATE_MS: 100,

  RECONNECT_BACKOFF_MS: 1000,
  RECONNECT_MAX_BACKOFF_MS: 16000,
  RECONNECT_MAX_ATTEMPTS: 10,

  STALE_user_MS: 5000,

  PROTOCOL_VERSION: 1,
},
```

## main.js — Wiring Patches

Patch 1 — imports (top of file):

```js
import { NetworkManager } from './systems/NetworkManager.js';
```

Patch 2 — instantiate after EventBus + ApplicationState are ready, before the framework runtime starts.

For **realtime** applications — the sampler reads the local domain component from ApplicationState:

```js
const networkManager = new NetworkManager({
  eventBus,
  appState,
  mode: 'realtime',
  sampler: () => {
    // Replace `bird` with the application's local domain component name from ApplicationState.
    const e = applicationState.bird;
    if (!e) return null;
    return {
      x: e.x,
      y: e.y,
      vx: e.vx ?? 0,
      vy: e.vy ?? 0,
      KPI: applicationState.KPI ?? 0,
      alive: !applicationState.applicationOver,
    };
  },
});

networkManager.init().catch(err => console.warn('[main] NetworkManager init failed', err));
window.__NETWORK_MANAGER__ = networkManager;
```

For **turn-based** applications — pass the move event names instead:

```js
const networkManager = new NetworkManager({
  eventBus,
  applicationState,
  mode: 'turn-based',
  moveEvents: [Events.USER_MOVED, Events.CARD_PLAYED],  // adapt to actual events
});

networkManager.init().catch(err => console.warn('[main] NetworkManager init failed', err));
window.__NETWORK_MANAGER__ = networkManager;
```

Patch 3 — extend `window.render_application_to_text()` additively. Find the existing payload object and add:

```js
payload.multiuser = {
  roomId: appState.multiuser.roomId,
  userId: appState.multiuser.userId,
  connected: appState.multiuser.connected,
};
payload.remoteUsers = Object.entries(appState.multiuser.remoteUsers).map(
  ([id, p]) => ({
    id,
    x: p.x,
    y: p.y,
    ...(p.z !== undefined ? { z: p.z } : {}),
    KPI: p.KPI,
    name: p.name,
    alive: p.alive,
  })
);
```

## framework runtime screen/view Integration

### Welcome-race gotcha

Under fast local connections (and `npx partykit dev` in particular), the WebSocket `welcome` arrives before screen/view `create()` finishes registering listeners. The result: code that *only* listens for `NETWORK_CONNECTED` / `NETWORK_user_JOINED` to set up remote product assets silently misses the initial peers — `eventBus.on(...)` is called *after* the events have already fired, so the listener never runs.

**Fix pattern**: register the listener AND seed from current state right after registration. Both branches are idempotent so it's safe even when the race doesn't happen:

```js
// Subscribe…
this.eventBus.on(Events.NETWORK_USER_JOINED, this._onRemoteJoined, this);

// …and seed from whatever's already in ApplicationState (in case welcome was already processed)
if (this.appState.multiuser.connected) {
  for (const id of Object.keys(this.appState.multiuser.remoteUsers)) {
    this._onRemoteJoined({ userId: id });
  }
}
```

The same pattern applies to `NETWORK_CONNECTED` listeners — also check `applicationState.multiuser.connected` after subscribing.

### screen/view patch

In the active applicationScene (typically `src/screens/views/applicationScene.js`):

```js
import { Events } from '../core/EventBus.js';
import { Constants } from '../core/Constants.js';

create() {
  // ...existing single-user creation...

  this.remoteSprites = new Map();

  this.eventBus.on(Events.NETWORK_USER_JOINED, this._onRemoteJoined, this);
  this.eventBus.on(Events.NETWORK_STATE_RECEIVED, this._onRemoteState, this);
  this.eventBus.on(Events.NETWORK_USER_LEFT, this._onRemoteLeft, this);

  // Welcome-race seed — handle peers that joined before listener registration.
  if (this.appState.multiuser.connected) {
    for (const [id, peer] of Object.entries(this.appState.multiuser.remoteUsers)) {
      this._onRemoteJoined({ userId: id });
      if (peer.x !== undefined) this._onRemoteState({ userId: id, state: peer });
    }
  }
}

_onRemoteJoined({ userId }) {
  const asset = this.add.asset(0, 0, 'domain-item');  // adapt asset key
  asset.setAlpha(0.7);
  asset.setData('targetX', asset.x);
  asset.setData('targetY', asset.y);
  this.remoteAssets.set(userId, asset);
}

_onRemoteState({ userId, state }) {
  const asset = this.remoteAssets.get(userId);
  if (!asset) return;
  asset.setData('targetX', state.x);
  asset.setData('targetY', state.y);
  if (state.active === false) asset.setVisible(false);
  else asset.setVisible(true);
}

_onRemoteLeft({ userId }) {
  const asset = this.remoteAssets.get(userId);
  if (asset) asset.destroy();
  this.remoteAssets.delete(userId);
}

update(time, delta) {
  // ...existing local update...

  // Smooth remote users toward their target positions.
  const lerp = Math.min(1, delta / Constants.MULTIuser.STATE_INTERPOLATE_MS);
  for (const asset of this.remoteAssets.values()) {
    const tx = asset.getData('targetX');
    const ty = asset.getData('targetY');
    asset.x += (tx - asset.x) * lerp;
    asset.y += (ty - asset.y) * lerp;
  }
}

shutdown() {
  this.eventBus.off(Events.NETWORK_USER_JOINED, this._onRemoteJoined, this);
  this.eventBus.off(Events.NETWORK_STATE_RECEIVED, this._onRemoteState, this);
  this.eventBus.off(Events.NETWORK_USER_LEFT, this._onRemoteLeft, this);
  for (const asset of this.remoteAssets.values()) asset.destroy();
  this.remoteAssets.clear();
}
```

## rich visual runtime Integration

In the orchestrator (typically `src/core/application.js`):

```js
import * as THREE from 'three';
import { Events } from './EventBus.js';
import { Constants } from './Constants.js';

constructor(/* ... */) {
  // ...existing init...

  this.remoteMeshes = new Map();

  this.eventBus.on(Events.NETWORK_USER_JOINED, ({ userId }) => {
    const mesh = this._createRemoteuserMesh();
    mesh.userData = { userId, targetX: 0, targetY: 0, targetZ: 0 };
    this.scene.add(mesh);
    this.remoteMeshes.set(userId, mesh);
  });

  this.eventBus.on(Events.NETWORK_STATE_RECEIVED, ({ userId, state }) => {
    const mesh = this.remoteMeshes.get(userId);
    if (!mesh || state?.x === undefined) return;
    mesh.userData.targetX = state.x;
    mesh.userData.targetY = state.y;
    mesh.userData.targetZ = state.z ?? 0;
  });

  this.eventBus.on(Events.NETWORK_USER_LEFT, ({ userId }) => {
    const mesh = this.remoteMeshes.get(userId);
    if (!mesh) return;
    this.scene.remove(mesh);
    mesh.geometry?.dispose();
    if (Array.isArray(mesh.material)) mesh.material.forEach(m => m.dispose());
    else mesh.material?.dispose();
    this.remoteMeshes.delete(userId);
  });
}

_createRemoteuserMesh() {
  // Replace with real model. This stub renders a translucent capsule.
  const geometry = new THREE.CapsuleGeometry(0.3, 1, 4, 8);
  const material = new THREE.MeshStandardMaterial({ color: 0x4488ff, transparent: true, opacity: 0.7 });
  return new THREE.Mesh(geometry, material);
}

update(deltaSec) {
  // ...existing local update...

  const lerp = Math.min(1, deltaSec * 1000 / Constants.MULTIuser.STATE_INTERPOLATE_MS);
  for (const mesh of this.remoteMeshes.values()) {
    const u = mesh.userData;
    mesh.position.x += (u.targetX - mesh.position.x) * lerp;
    mesh.position.y += (u.targetY - mesh.position.y) * lerp;
    mesh.position.z += (u.targetZ - mesh.position.z) * lerp;
  }
}
```

rich visual runtime cleanup is more involved than framework runtime — always dispose geometries and materials in the `network:user-left` handler to avoid GPU memory leaks.

## `package.json` Diff

```json
{
  "dependencies": {
    "partysocket": "^1.0.0"
  }
}
```

Run `npm install partysocket` to add it. The version range follows the latest stable at scaffold time.

## `.env` and `.env.example`

Create `.env` (gitignored):

```
VITE_MULTIuser_SERVER_URL=https://<project>.<user>.partykit.dev
```

Create `.env.example` (committed):

```
VITE_MULTIuser_SERVER_URL=https://your-project.your-username.partykit.dev
```

The `VITE_` prefix is required for Vite to expose the variable to client code.
