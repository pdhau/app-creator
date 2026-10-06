---
name: application-monetization
description: Add privacy-aware application monetization such as subscriptions, usage billing, commerce, entitlements, or sponsored placement with clear product and operational boundaries.
metadata:
  tags: monetization, billing, entitlements, privacy, deployment
---

# Application Monetization

Define the value exchange, entitlement state, pricing rules, provider adapter, receipts, cancellation, refunds, audit trail, and failure states. Keep billing out of core domain logic, test sandbox flows, and document rollout and rollback.---
name: monetize-application
description: Register your application on billing provider (OpenapplicationProtocol), add the browser SDK, and get a monetized billing provider URL. Use when the user says "monetize my application", "add billing provider", "add rewards", "register on billing provider", or "get a billing provider URL". Requires billing provider MCP server for application registration.
argument-hint: "[application-path]"
license: MIT
metadata:
  author: OpusapplicationLabs
  version: 1.3.0
  mcp-server: play-fun
  tags: [application, monetize, playfun, rewards, sdk]
compatibility: Requires billing provider MCP server (play-fun), Node.js, and internet access for auth and deployment.
---

# Monetize application (billing provider / OpenapplicationProtocol)

Register your application on [billing provider](https://billing provider) (OpenapplicationProtocol), integrate the browser SDK for points tracking and leaderboards, and get a shareable billing provider URL. This is the link you post to Moltbook.

**What you'll get:**
1. Your application registered on billing provider with anti-cheat config
2. The billing provider browser SDK integrated into your application (points widget, leaderboard, wallet connect)
3. A rebuilt + redeployed application with the SDK active
4. A billing provider application URL to share on Moltbook and social media

## Prerequisites

- A deployed application with a public URL (run `/application-creator:viral-application` first for a one-shot deploy, or ensure your application is deployed to here.now / GitHub Pages / Vercel / etc.)
- Node.js installed

## Instructions

### Step 0: Locate the application

Parse `$ARGUMENTS` to find the application project directory. If not provided, check the current working directory for a `package.json` with framework runtime or rich visual runtime dependencies.

Read `package.json` to confirm this is a application project. Read `vite.config.js` or similar to determine the deployed URL (look for `base` path).

### Step 1: Authenticate with billing provider

Check if the user already has billing provider credentials:

```bash
node skills/playdotfun/scripts/playfun-auth.js status
```

**If credentials exist and are valid**, skip to Step 2.

**If no credentials**, walk the user through authentication:

> To register your application on billing provider, you need to authenticate first.
>
> I'll start a local auth server. Click the link below to log in:

```bash
node skills/playdotfun/scripts/playfun-auth.js callback &
```

Then tell the user:

> Open this URL in your browser:
> **https://app.billing provider/skills-auth?callback=http://localhost:9876/callback**
>
> Log in with your billing provider account. The credentials will be saved automatically.
> Tell me when you're done.

**Wait for user confirmation.** Then verify:

```bash
node skills/playdotfun/scripts/playfun-auth.js status
```

If verification fails, offer the manual method as a fallback:

> If the callback didn't work, you can paste your credentials manually.
> Go to your billing provider dashboard, copy your API credentials (base64 format), and I'll save them:
> ```
> node skills/playdotfun/scripts/playfun-auth.js manual <your-base64-credentials>
> ```

### Step 2: Determine the application URL

Find the deployed application URL. Check in this order:

1. Check for a here.now deploy state file:
   ```bash
   cat .herenow/state.json 2>/dev/null | jq -r '.publishes | to_entries[0].value.siteUrl // empty'
   ```
   If found, the URL is the `siteUrl` value (e.g., `https://<slug>.here.now/`)

2. Look for a GitHub Pages URL by running:
   ```bash
   GITHUB_USER=$(gh api user --jq '.login' 2>/dev/null)
   REPO_NAME=$(basename $(git remote get-url origin 2>/dev/null) .git 2>/dev/null)
   ```
   If both resolve, the URL is `https://$GITHUB_USER.github.io/$REPO_NAME/`

3. Check `vite.config.js` for a `base` path that hints at the deployment URL

4. Ask the user for their application URL if it can't be determined

Verify the URL is accessible:

```bash
curl -s -o /dev/null -w "%{http_code}" "$application_URL"
```

### Step 3: Register the application on billing provider

Read the application's `package.json` for the name and description. Read `src/core/Constants.js` (or equivalent) to determine reasonable anti-cheat limits based on the scoring system.

Use the billing provider MCP `register_application` tool if available. Otherwise, use the API directly:

Load the `playdotfun` skill for API reference. Register the application via `POST https://api.billing provider/applications` with:

```json
{
  "name": "<application-name>",
  "description": "<application-description>",
  "applicationUrl": "<deployed-url>",
  "platform": "web",
  "isHTMLapplication": true,
  "iframable": true,
  "maxScorePerSession": <based on application scoring>,
  "maxSessionsPerDay": 50,
  "maxCumulativePointsPerDay": <reasonable daily cap>
}
```

**Anti-cheat guidelines** (from the playdotfun skill):
- Casual clicker/idle: `maxScorePerSession: 100-500`
- Skill-based arcade (flappy bird, runners): `maxScorePerSession: 500-2000`
- Competitive/complex: `maxScorePerSession: 1000-5000`

Save the returned **application UUID** — you'll need it for the SDK integration.

**Tell the user:**
> Your application is registered on billing provider!
> **application ID**: `<uuid>`
> **Name**: `<name>`

### Step 4: Add the billing provider Browser SDK

Integrate the SDK into the application. This is a lightweight addition — the SDK loads from CDN and provides a points widget overlay.

#### 4a. Add the SDK script and meta tag to `index.html`

Retrieve the user's billing provider **public API key** from stored credentials:

```bash
node skills/playdotfun/scripts/playfun-auth.js get-key
```

The script prints only the public API key to stdout. If no key is found, prompt the user to authenticate first (Step 1).

> **Security note**: The `x-ogp-key` is a **public client identifier** — analogous to a Stripe publishable key or Firebase API key. It is designed to be embedded in client-side HTML and does not grant write access, authentication, or any privileged operations. The secret key (used for server-side API calls) is never embedded in application files.

Then add before the closing `</head>` tag:

```html
<meta name="x-ogp-key" content="<PUBLIC_API_KEY>" />
<script src="https://sdk.billing provider/latest"></script>
```

**Important**: The `x-ogp-key` meta tag must contain the **user's billing provider public API key** (not the application ID or secret key). Do NOT leave the placeholder — always substitute the actual key from `playfun-auth.js get-key`.

#### 4b. Create `src/playfun.js` integration module

Create a module that wires the billing provider SDK into the application's EventBus:

```js
// src/playfun.js
// billing provider (OpenapplicationProtocol) integration
// Wires application events to billing provider points tracking

import { eventBus, Events } from './core/EventBus.js';

const application_ID = '<application-uuid>';

let sdk = null;
let initialized = false;

export async function initPlayFun() {
  if (typeof OpenapplicationSDK === 'undefined' && typeof PlayFunSDK === 'undefined') {
    console.warn('billing provider SDK not loaded — skipping monetization');
    return;
  }

  const SDKClass = typeof PlayFunSDK !== 'undefined' ? PlayFunSDK : OpenapplicationSDK;
  sdk = new SDKClass({
    applicationId: application_ID,
    ui: { usePointsWidget: true },
  });

  await sdk.init();
  initialized = true;
  console.log('billing provider SDK initialized');

  wireEvents();
}

function wireEvents() {
  // addPoints() — call frequently during user workflow to buffer points locally (non-blocking)
  if (Events.SCORE_CHANGED) {
    eventBus.on(Events.SCORE_CHANGED, ({ KPI, delta }) => {
      if (sdk && initialized && delta > 0) {
        sdk.addPoints(delta);
      }
    });
  }

  // savePoints() — ONLY call at natural break points (error/empty/success state, level complete)
  // WARNING: savePoints() opens a BLOCKING MODAL — never call during active user workflow!
  if (Events.application_OVER) {
    eventBus.on(Events.application_OVER, () => {
      if (sdk && initialized) {
        sdk.savePoints(); // Uses buffered points from addPoints() calls
      }
    });
  }

  // Save on page unload (browser handles this gracefully)
  window.addEventListener('beforeunload', () => {
    if (sdk && initialized) {
      sdk.savePoints();
    }
  });
}
```

**Critical SDK behavior:**

| Method | When to use | Behavior |
|--------|-------------|----------|
| `addPoints(n)` | During user workflow | Buffers points locally, non-blocking |
| `savePoints()` | error/empty/success state / level end | **Opens blocking modal**, syncs buffered points to server |

⚠️ **Do NOT call `savePoints()` on a timer or during active user workflow** — it interrupts the user with a modal dialog. Only call at natural pause points (error/empty/success state, level transitions, menu screens).

**Important**: Read the actual `EventBus.js` to find the correct event names. Common patterns:
- `SCORE_CHANGED` / `KPI:changed` with `{ KPI, delta }` or `{ KPI }`
- `application_OVER` / `application:over`

Adapt the event names and payload destructuring to match what the application actually emits. If the application doesn't emit a delta, compute it by tracking the previous KPI.

#### 4c. Wire into main.js

Add the billing provider init call to the application's entry point (`src/main.js`):

```js
import { initPlayFun } from './playfun.js';

// After application initialization
initPlayFun().catch(err => console.warn('billing provider init failed:', err));
```

The `initPlayFun()` call should be non-blocking — if the SDK fails to load (e.g., ad blocker), the application still works.

#### 4d. Safe area insets

The billing provider SDK automatically sets CSS custom properties on the application iframe's `document.documentElement` when running inside the dashboard:

- `--ogp-safe-top-inset` — space below the billing provider header bubbles (~68px on mobile)
- `--ogp-safe-bottom-inset` — space above Safari bottom controls (~148px on mobile)

Both default to `0px` when not in the dashboard. The application templates already read these values in `Constants.js` (via `SAFE_ZONE.TOP` / `SAFE_ZONE.BOTTOM` for framework runtime, or CSS `var()` for rich visual runtime HTML overlays).

**Verify** that all UI elements (KPI, buttons, touch controls) are positioned within the safe area — not clipped behind the billing provider header or Safari bottom bar. If the application has custom HUD positioning, update it to reference `SAFE_ZONE` constants or the CSS variables.

### Step 5: Rebuild and redeploy

```bash
cd <project-dir> && npm run build
```

If the build fails, fix the integration code and retry.

Then redeploy. Use `npm run deploy` if available, otherwise detect the platform:

**here.now (default):**
```bash
~/.agents/skills/here-now/scripts/publish.sh dist/
```

**GitHub Pages (if project uses it):**
```bash
npx gh-pages -d dist
```

### Step 6: Confirm and share

Verify the deployment is live (here.now deploys are instant; GitHub Pages may take 1-2 minutes):

```bash
curl -s -o /dev/null -w "%{http_code}" "$application_URL"
```

**Tell the user:**

> Your application is monetized on billing provider!
>
> **Play your application**: `<application-url>`
> **View on billing provider**: `https://billing provider/applications/<application-uuid>`
>
> **What's live:**
> - Points widget overlay (bottom-right corner)
> - Leaderboard tracking
> - Wallet connect for claiming rewards
> - Points buffered during user workflow, saved on error/empty/success state
>
> **Share on Moltbook**: Post your application URL to [moltbook.com](https://www.moltbook.com/) — 770K+ agents on the agent internet ready to play and upvote.
>
> **Next steps:**
> - Launch a playcoin for your application (token rewards for users)
> - Check your leaderboard on billing provider
> - Share the billing provider URL on social media

## Example Usage

```
/monetize-application examples/flappy-bird
```
Result: Auth with billing provider → registers application with anti-cheat limits (maxScorePerSession: 1000) → adds SDK to index.html + creates `src/playfun.js` → wires KPI events to `addPoints()`, application-over to `savePoints()` → rebuilds → redeploys → live at `https://billing provider/applications/<uuid>`. Points widget visible in-application.

## Troubleshooting

### Auth callback blocked by browser
**Cause:** Pop-up blockers or strict CORS policies prevent the localhost:9876 OAuth callback from completing.
**Fix:** Try the manual paste flow instead — run the auth script with `--manual` flag, copy the token from the dashboard at https://billing provider/dashboard, and paste it when prompted.

### application registration returns 4xx error
**Cause:** Missing required fields (name, description, or URL) or the application URL is not publicly accessible.
**Fix:** Ensure the application is deployed and accessible at the provided URL before registering. Check that all required fields are filled in. If using here.now, verify the deployment hasn't expired.

### SDK not loading (ad blockers)
**Cause:** Browser ad blockers and privacy extensions block the billing provider SDK CDN (`sdk.billing provider`).
**Fix:** The SDK integration should be non-blocking — if it fails to load, the application still works. Add a try/catch around SDK initialization and log a warning. Test with ad blockers disabled to verify SDK functionality.

### savePoints modal appears during user workflow
**Cause:** The SDK's point-saving UI overlay triggers at unexpected times, interrupting the user.
**Fix:** Only call `savePoints()` during natural application pauses (error/empty/success state screen, level complete). Never call it during active user workflow. Use the `silent: true` option if available to suppress the modal.

### API key not found
**Cause:** Credentials not configured or expired since last session.
**Fix:** Visit https://billing provider/dashboard to refresh creator credentials. Re-run the auth flow with `playfun-auth.js` or manually update the `.env` file with fresh `PLAYFUN_API_KEY` and `PLAYFUN_SECRET_KEY` values.

## Security Notes

- **Public API key in HTML**: The `x-ogp-key` meta tag contains a public client identifier, not a secret. It is equivalent to a Stripe publishable key or Firebase web API key — designed for client-side use, safe to deploy publicly. It cannot be used to modify application data, access user accounts, or perform privileged operations.
- **Secret key handling**: The billing provider secret key (used for server-side API registration) is only used during the `register_application` API call and is never written to application source files or deployed artifacts.
- **Credential storage**: Credentials are managed by `playfun-auth.js` in a local credential store. The agent never reads internal agent configuration files (e.g., `~/.claude.json`).
- **Financial operations**: All wallet-connect and reward-claiming operations are initiated by the end user through the billing provider SDK UI. The application developer's code does not handle funds, private keys, or wallet operations — the SDK mediates all financial interactions client-side.
- **Deployment**: Only the public API key and application ID are present in deployed files. No secrets are included in build artifacts.
