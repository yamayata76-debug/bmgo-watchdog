# bmgo-watchdog — Telegram alerts for attacks, outages and cloak leaks ($0)

Public repo on purpose: GitHub Actions is unlimited-free on public repos, and
this repo contains **zero code and zero secrets** — only this workflow, which
reads everything from Actions secrets. Nothing here reveals the host, the
owner, or the stack.

## What it watches (every 10 min)

| Check | Alert text | Meaning |
|---|---|---|
| `SITE_URL/api/health` | `site-down` | Visitors see downtime — flip origin NOW |
| `ORIGIN_A_URL`, `ORIGIN_B_URL` | `originA-down` / `originB-down` | One backend dead/suspended |
| `Server:` header == cloudflare | `proxy-leak` | Proxy disabled or bypassed — origin stack visible |
| CSP header present | `csp-missing` | Middleware/deploy broken |
| Authoritative CNAME == expected | `dns-drift(...)` | DNS changed behind your back (hijack or stray edit) |

Alerts fire **only on change** (new failure or recovery), plus a run link.
Quiet when green.

## Setup (one time, ~5 min of pasting)

Repo Settings → Secrets and variables → Actions → New repository secret:

| Secret | Value |
|---|---|
| `SITE_URL` | `https://blockmango.shop` |
| `DOMAIN` | `blockmango.shop` |
| `ORIGIN_A_URL` | `https://<your>.leapcell.dev` (no trailing slash) |
| `ORIGIN_B_URL` | `https://<your>.onrender.com` (no trailing slash) |
| `EXPECTED_TARGET` | current CNAME target, e.g. `<your>.leapcell.dev` (bare host, no scheme). **Update on every flip** (failover.ps1 reminds you; it can sync this automatically via `gh secret set` where gh is authed) |
| `TG_BOT_TOKEN` | Telegram bot token for alerts |
| `TG_CHAT_ID` | your Telegram user/chat id |
| `CF_API_TOKEN` | Cloudflare "Edit zone DNS" token (read is enough for the drift check) |

Then Actions → watchdog → Run workflow (manual test run). Break something on
purpose once (e.g. wrong EXPECTED_TARGET) to see the Telegram message arrive.
