# LLM-OVERVIEW — vig
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
Vig (`vig`) is a Cloudflare-native sports prediction pool platform supporting multiple pool types (wins pools, async snake drafts, brackets, squares, and spread pools) across major sports leagues (NBA, NFL, NCAA). The core application is built as an SSR web application using Astro 5 in server mode (`output: 'server'`) deployed to Cloudflare Pages (`https://khamel.com`), backed by a Cloudflare D1 SQLite database (`vig-db`), Cloudflare KV (`vig-kv` / `KV` / `SESSION`), and scheduled background sync Cloudflare Workers (`vig-sync`, `vig-standings`).

## Machine & Host Ownership
Production services run serverlessly on Cloudflare edge infrastructure (Cloudflare Pages project `vig` and Cloudflare Workers `vig-sync` and `vig-standings`); `homelab.yaml` records production host runtime as managed by Cloudflare. Development and orchestration tasks run locally or dispatch via SSH to homelab hosts `oci-ts` and `macmini-ts` per `AGENTS.md`, with web search routing to the homelab Argus search service on `http://100.112.130.100:8270`.

## What is actually built
- **Frontend App (`src/pages/`, `src/layouts/`, `src/components/`)**: Astro 5 SSR pages using Tailwind CSS. React integrations were removed (`astro.config.mjs`) due to Cloudflare Workers `MessageChannel` incompatibility.
  - *Public & Authentication Pages*: Landing page (`index.astro`), authentication pages (`login.astro`, `register.astro`), and invite code handler (`join.astro`).
  - *User Dashboards & Pool Views*: User pool and debt dashboard (`dashboard.astro`), custom NBA 2026 pool view (`nba26.astro`), dynamic pool details and pick selection (`[slug]/index.astro`), and real-time async snake draft interface (`drafts/[id].astro`).
  - *Administration*: Global admin control panel (`admin/index.astro`), payment auditing (`admin/payments.astro`), and pool management (`pools/[id]/manage.astro`).
  - *UI Components*: `DashboardHeader.astro`, `MyPools.astro`, `MyPicks.astro`, `UpcomingGames.astro`, `PaymentCard.astro`, `PaymentModal.astro`, `DebtsList.astro`, and `CreatePoolWizard.astro`.
- **API Endpoints (`src/pages/api/`)**:
  - `auth/`: Local authentication (`login.ts`, `register.ts`, `logout.ts`, `me.ts`), Google OAuth2 initiation (`google.ts`), callback processing (`google-callback.ts`), and account setup completion (`google-complete.ts`).
  - `events/`: Pool listing (`index.ts`), pool details (`[slug].ts`), user selections (`[slug]/selections.ts`), and payment tracking (`[slug]/payments.ts`).
  - `pools/` & `templates/`: Instantiating pools from templates (`pools/create-from-template.ts`), pool status lifecycle operations (`pools/[id]/lifecycle.ts`), and template CRUD operations (`templates/index.ts`).
  - `drafts/`: Draft creation (`create.ts`), turn/state status monitoring (`[id]/status.ts`), and submitting draft picks (`[id]/pick.ts`).
  - `payments/` & `debts.ts`: Entry fee submission (`submit.ts`), admin payment confirmation (`confirm.ts`), payment status listing (`list.ts`), and user P2P debt ledger management.
  - `admin/`: Manual score sync trigger (`sync-scores.ts`), manual scoring calculation (`scoring.ts`), team seeding (`seed-teams.ts`), and payment reminder dispatch (`payment-reminders.ts`).
  - `invites.ts` & `dashboard.ts`: Invite code generation/validation and user dashboard data aggregations.
  - `ws/leaderboard.ts`: Real-time leaderboard handler.
- **Backend Libraries (`src/lib/`)**:
  - `auth.ts` & `oauth-google.ts`: Web Crypto password hashing, JWT creation and validation, session cookies, and Google OAuth2 integration flow.
  - `db.ts` & `middleware.ts`: D1 database query wrapper and Astro request middleware injecting authenticated user sessions into execution context.
  - `pools.ts`, `invites.ts`, `drafts.ts`, `draft-notifications.ts`: Pool management, invite processing, snake draft state machine (round calculation, pick ordering, turn deadlines), and draft turn notifications.
  - `payments.ts` & `debts.ts`: Entry fee tracking, prize distribution settings (Venmo/CashApp instructions), and peer-to-peer debt ledgers.
  - `scoring.ts`: Automated standings computation and D1 standings table rank updating.
  - Data Ingestion Layer (`sports-api.ts`, `espn-api.ts`, `api-sports.ts`, `playwright-scraper.ts`, `sports-reference-scraper.ts`, `standings-scraper.ts`): Multi-source standings integration layer using public ESPN API, Sports-Reference HTML scraping, API-Sports, and Playwright fallback browser scraping.
  - `email.ts`: Resend API integration for email alerts.
  - `leaderboard-do.ts`: Cloudflare Durable Object implementation for live leaderboard state.
- **Database Schema (`vig-db` D1 SQLite, `migrations/`)**:
  - Core User & Pool Tables: `users` (with OAuth support), `events` (pools), `options` (selectable teams/players), `selections` (user choices), `games` (match scores), `standings` (materialized ranks), and `sessions`.
  - Financial & Invites: `payment_settings`, `payments`, `debts`, `pool_templates`, `invite_codes`, and `event_participants`.
  - Snake Draft Infrastructure: `drafts`, `draft_picks`, `draft_timers`, and `draft_settings`.
  - External Data Cache: `espn_standings`.
- **Background Cron Workers & Scripts (`scripts/`, `wrangler.*.toml`)**:
  - `vig-sync` (`scripts/sync-nba26.js`, `wrangler.sync.toml`): Cloudflare Cron Worker executing every 3 hours (`0 */3 * * *`) to fetch NBA game data via The Rundown API and update D1.
  - `vig-standings` (`scripts/sync-standings.ts`, `wrangler.standings.toml`): Cloudflare Cron Worker executing twice daily (9am & 11pm ET: `0 13 * * *`, `0 3 * * *`) updating league standings in D1.
  - Local CLI Tools: `scripts/scrape-current-standings.js` (manual standings scraping) and `scripts/test-espn-api.js` (ESPN API diagnostic utility).

## Canonical entry points
- `npm run dev`: Launch local Astro development server with platform proxies for D1 database and KV bindings.
- `npm run build`: Compile Astro SSR application to `dist/`.
- `npm run deploy`: Build project and deploy output to Cloudflare Pages (`wrangler pages deploy dist --project-name=vig`).
- `npm run db:migrate`: Run database schema migrations against D1 (`wrangler d1 execute vig-db --file=migrations/0001_initial.sql`).
- `npm run db:console`: Interactive Cloudflare D1 database CLI console (`wrangler d1 execute vig-db --command`).
- `wrangler deploy -c wrangler.sync.toml`: Deploy the `vig-sync` background game score sync worker to Cloudflare Workers.
- `wrangler deploy -c wrangler.standings.toml`: Deploy the `vig-standings` background standings worker to Cloudflare Workers.
