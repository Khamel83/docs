# LLM-OVERVIEW — poytz
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
Personal cloud infrastructure and URL redirector running on Cloudflare Workers (`khamel-redirector` package, project ID `poytz`). Routes external HTTP traffic using HTTP 307 redirects to internal homelab services exposed via Tailscale Funnel (`https://homelab.deer-panga.ts.net`). Features an OAuth auth proxy, cross-device clipboard sync, paste text sharing, webhook storage, status monitoring, Home Assistant triggers, and periodic maintenance tasks designed for Cloudflare Workers free tier ($0/month).

## Machine & Host Ownership
Runs as a serverless worker on Cloudflare Workers edge network; target endpoints route to a Tailscale Funnel host (`homelab.deer-panga.ts.net`). No specific local physical machine or multi-host server matrix is declared in repo evidence.

## What is actually built
- **Worker Router (`src/index.js`)**: ES module exporting standard `fetch` HTTP router and `scheduled` cron handler.
  - **Redirect Engine**: Returns HTTP 307 redirects to preserve HTTP POST request bodies and query parameters when forwarding to Tailscale Funnel.
  - **Root Proxy & Dashboard**: `handleRootProxy` routes `/` to the Homepage dashboard with OAuth SSO (`AUTHORIZED_EMAIL` configured as `zoheri@gmail.com`, username `khamel`).
  - **Auth Proxy**: OAuth callback handling with redirects to root (`/`).
  - **Public & Route Management API**: KV-backed endpoints for link shortener and dynamic route management.
  - **Services**: Webhook receiver for payload capture, cross-device clipboard sync, text paste sharing with short URLs, system status health probes, and Home Assistant trigger endpoints (`Home API`).
  - **Cron Task Handler**: Background jobs running health checks, data cleanup, and KV write throttling to keep write operations within free-tier caps.
- **Homelab Contract (`homelab.yaml`)**: Spec `homelab.project/v1` binding project `poytz`, lifecycle `development`, monitoring state `standby`, and event sink `homelab-api:/observations`.
- **Worker Configuration & Build (`package.json`, `wrangler.toml`)**: Uses `wrangler` (`^3.0.0`) for deployment (`npm run deploy`), local dev (`npm run dev`), and log streaming (`npm run tail`).

## Canonical entry points
- `src/index.js`: Main Cloudflare Worker router and cron handler entry point.
- `wrangler.toml`: Worker runtime, KV namespace, and environment configuration.
- `package.json`: Project scripts (`deploy`, `dev`, `tail`) and dependencies.
- `homelab.yaml`: Homelab project contract and metadata.
- `README.md`: Architectural documentation and project overview.
- `OPERATIONS.md`: Operations and API troubleshooting guide.
