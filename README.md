# ⚡ Mission Control — OpenClaw Dashboard

A zero-dependency, single-file web dashboard for monitoring and managing your OpenClaw Gateway.

## Features

- **Command Center** — System health, active agents, queue depth, connector status, failures, running sessions, scheduled jobs, token usage
- **Task Board** — Kanban view of all sessions across 8 columns (Inbox → Archived)
- **Mission Board** — Parent/child session tree showing subagent hierarchies
- **Run Detail** — Searchable session list with expandable details (prompt, timeline, tool calls, outputs, errors)
- **Auto-refresh** every 15 seconds with live "last updated" counter
- **Ed25519 device identity** — automatic keypair generation, challenge signing, and device pairing
- **Dark theme** optimized for monitoring
- **Fully offline** — no CDN, no build step, no dependencies

## Quick Start

### 1. Get Your Gateway URL and Token

On the machine running your OpenClaw gateway:

```bash
# Get your gateway auth token
openclaw config get gateway.auth.token

# Your gateway URL is typically:
# wss://your-server:18789
# or ws://localhost:18789 for local access
```

### 2. Open the Dashboard

Open `index.html` in any modern browser (Chrome, Firefox, Safari, Edge).

### 3. Connect

1. Enter your **Gateway URL** (e.g., `wss://your-server:18789`)
2. Enter your **Token**
3. Click **Connect**

### 4. Handle Device Pairing

On first connect, the gateway requires device approval:

1. You'll see a yellow banner: *"⚠️ Pairing Required"*
2. On your gateway host, run:
   ```bash
   openclaw devices approve <device-id>
   ```
3. Click **Reconnect** in the dashboard

Once paired, the device token is stored and future connections are automatic.

## Hosting on GitHub Pages

1. Create a GitHub repository
2. Add `index.html` to the root (or a `/docs` folder)
3. Go to **Settings → Pages**
4. Set source to your branch and folder
5. Access at `https://yourusername.github.io/repo-name/`

> **Note:** Your gateway must be accessible from the internet (or via a tunnel) for GitHub Pages hosting to work. The token is stored in your browser's `localStorage` and never sent to GitHub.

## How It Works

### Connection Flow

1. Browser opens WebSocket to your gateway
2. Gateway sends a challenge (nonce + timestamp)
3. Browser generates/loads an Ed25519 keypair (stored in `localStorage`)
4. Browser signs the challenge with v3 protocol payload
5. Browser sends `connect` request with signature + auth token
6. Gateway responds with `hello-ok` (success) or `PAIRING_REQUIRED`
7. On success, a device token is stored for future auto-reconnect

### Data Sources

| RPC Method | Used For |
|---|---|
| `status` | System health, uptime, token usage |
| `health` | Channel connector status |
| `sessions.list` | All session data (kanban, missions, runs) |
| `cron.list` | Scheduled jobs |
| `system-presence` | Connected clients/agents |
| `models.list` | Available AI models |
| `node.list` | Paired compute nodes |

### Auto-Reconnect

If the WebSocket drops, the dashboard automatically reconnects with exponential backoff (1s → 2s → 4s → 8s → 30s max).

## Browser Requirements

- Chrome 113+, Edge 113+, or any Chromium-based browser (Ed25519 WebCrypto support)
- Firefox 128+ (Ed25519 support added in 2024)
- Safari 17+ (partial — Ed25519 may require flag)

Ed25519 in WebCrypto is required. If your browser doesn't support it, the dashboard will show a crypto error.

## Settings

Click the ⚙️ gear icon in the navbar to:
- Update gateway URL or token
- **Reset All Data** — clears stored credentials, keypair, and device token

## Security Notes

- Your gateway token is stored only in `localStorage` in your browser
- The Ed25519 private key never leaves your browser
- No data is sent to any third party
- For production use, always use `wss://` (TLS) rather than `ws://`

## License

MIT
