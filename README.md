# Mailtracku

> Gmail email tracker as a Chrome extension: logs every mail you send, tracks opens and link clicks,
> sends follow-up reminders, and exports everything to Excel.

**Version:** 1.10.0 | **Author:** Bhushan Bambarkar | **Platform:** Chrome (Manifest V3) + Gmail

## Features

- **Auto-log** every Gmail mail you send, with reply detection
- **Open tracking** via a tracking pixel, with desktop notifications
- **Link-click tracking** via redirects, with notifications
- **Follow-up reminders** when a mail stays unopened for N hours (Snooze 24 h / Dismiss)
- **Excel export**: sheets for Email Tracking, Tracking Events and Summary; auto-updating file on paid plans
- **Dashboard** with best-time chart and recent activity
- **Self-open filtering** (3 layers) so your own views are not counted
- **Privacy mode**
- **Plans:** Free, Silver (Rs 5), Gold (Rs 10), Platinum (Rs 15) with signed license keys (Ed25519)

## Repository layout

| Folder | Description |
|---|---|
| `extension/` | Chrome extension (background service worker, content script for Gmail, popup, Excel builder) |
| `tracking-server/` | Node 18+ server (no dependencies) that serves the open pixel and click redirects |
| `license-server/` | Node 18+ license server that issues and verifies plan keys |
| `website/` | Landing page with pricing and "Check my plan" (`index.html`) |

## Architecture

```
Gmail tab --content.js--> background.js --polls /events every 30 s--> tracking-server
   |                          |                                          ^
   | wraps links, adds pixel  | IndexedDB (mt_tracking)                  | recipient opens mail / clicks link
   v                          v
 recipient  ----------------------------------------------------------->
                popup.js (dashboard) --> xlsxbuilder.js (Excel)
extension <--signed token--> license-server <--- website (/license/status)
```

## Getting started

1. **Tracking server**
   ```bash
   cd tracking-server
   API_KEY="a-long-random-secret" PORT=8080 node server.js
   ```
   Put it behind HTTPS (Cloudflare Tunnel, Render, Fly.io, Railway, or a VPS with Caddy/nginx).
2. **Extension:** open `chrome://extensions`, enable Developer mode, click **Load unpacked**, choose `extension/`.
3. **Connect:** extension popup, then *Tracking & reminders settings*, enter server URL and API key, then *Save & test connection*.
4. **License server (seller only):** see `license-server/README.md`. Until `MT_LICENSE_URL` and `MT_PUBLIC_KEY` are set in `extension/plans.js`, all features stay unlocked.

## Plans

| | Free | Silver | Gold | Platinum |
|---|---|---|---|---|
| Price | Rs 0 | Rs 5 | Rs 10 | Rs 15 |
| Tracked mails / month | 0 | 50 | 300 | Unlimited |
| Open tracking + notifications | - | Yes | Yes | Yes |
| Follow-up reminders | - | 48 h / 72 h | Any | Any |
| Link-click tracking | - | - | Yes | Yes |
| Auto-updating Excel | - | - | Yes | Yes |
| Best-time chart + activity feed | - | - | Yes | Yes |
| Email IDs | 1 | 1 | 3 | Unlimited |
| Browsers per key | - | 1 | 2 | 3 |

Always free: auto-log, reply detection, manual Excel export, privacy mode.

## Known limitations

- Gmail loads images through Google's proxy, so opens may show Google IPs or count on preview.
- Apple Mail Privacy Protection and corporate scanners can cause false opens/clicks.
- With multiple recipients the pixel is shared.
- Notifications arrive within about 30-60 s and Chrome must be running.
- The extension code is readable, so license checks stop casual tampering only.

## Tech stack

JavaScript (ES2020), Chrome Extension Manifest V3, IndexedDB, Node.js 18+, ExcelJS, Ed25519 signatures (Node `crypto`).

## Security

Never commit API keys, `ADMIN_KEY`, or any `data/` folder (the license server's `keys.json` holds the private signing key).

## Folder names in the sub-READMEs

The sub-READMEs were written for the original zip. Mapping: `Mailtracku/` is `extension/`,
`Mailtracku-server/` is `tracking-server/`, `Mailtracku-license/` is `license-server/`.

## License

Add a `LICENSE` file (MIT or proprietary) before making the repo public.
