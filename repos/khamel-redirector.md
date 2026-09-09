# LLM-OVERVIEW — khamel-redirector
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
A personal URL shortener and public HTTP traffic router deployed as a Cloudflare Worker. It intercepts public requests under `khamel.com/*` and executes 302 redirects to internal homelab, cloud (OCI-Dev), and local (Mac Mini) endpoints published via Tailscale Funnel (`*.deer-panga.ts.net`), as well as external web destinations. It replaces traditional reverse proxies (Traefik, Nginx), manual SSL certificate handling, and complex DNS configurations.

## Machine & Host Ownership
No multi-host execution information is available; runs as a serverless Cloudflare Worker (`host: unknown` in `homelab.yaml`) routing requests to Tailscale Funnel endpoints on `homelab`, `oci-dev`, and `omars-mac-mini` target hosts.

## What is actually built
- **Worker Runtime Core (`src/index.js`)**: Single-file Cloudflare Worker script that inspects incoming URL request paths against a static in-memory mapping object (`ROUTES`) and outputs HTTP 302 redirect responses.
- **Route Registry (`ROUTES` mapping)**:
  - `fd` → `https://homelab.deer-panga.ts.net/frontdoor/` (Idea refinement service)
  - `jellyfin` → `https://homelab.deer-panga.ts.net/jellyfin/` (Media streaming)
  - `requests` → `https://homelab.deer-panga.ts.net/requests/` (Jellyseerr media requests)
  - `tv` → `https://homelab.deer-panga.ts.net/tv/` (Sonarr TV shows)
  - `movies` → `https://homelab.deer-panga.ts.net/movies/` (Radarr movies)
  - `music` → `https://homelab.deer-panga.ts.net/music/` (Lidarr music)
  - `docs` → `https://homelab.deer-panga.ts.net/docs/` (Paperless document management)
  - `photos` → `https://homelab.deer-panga.ts.net/photos/` (Immich photo library)
  - `recipes` → `https://homelab.deer-panga.ts.net/recipes/` (Mealie recipe manager)
  - `code` → `https://homelab.deer-panga.ts.net/code/`
- **Homelab Project Contract (`homelab.yaml`)**: `homelab.project/v1` schema contract designating project ID `khamel-redirector`, service lifecycle `development`, monitoring state `standby`, runtime unit `khamel-redirector`, and event sink `homelab-api:/observations`.
- **Project Configuration & Deployment**:
  - `package.json`: Package declaration with `wrangler` (^3.0.0) devDependency.
  - `wrangler.toml`: Cloudflare Wrangler CLI configuration file.
  - `MINIMAL.md`: Project dependency structure and cost breakdown documentation.

## Canonical entry points
- `src/index.js` — Cloudflare Worker entry point and routing table definitions
- `wrangler.toml` — Cloudflare deployment and worker runtime configuration
- `homelab.yaml` — Homelab contract and project metadata declaration
- `npm run dev` — Launch local development worker emulator (`wrangler dev`)
- `npm run deploy` — Deploy updated redirector Worker to Cloudflare edge (`wrangler deploy`)
- `npm run tail` — Stream live production log events (`wrangler tail`)
