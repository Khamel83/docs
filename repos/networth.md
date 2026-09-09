# LLM-OVERVIEW — networth
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
East Side LA women's tennis ladder system with automated monthly opponent pairings, games-won rankings, player availability tracking, self-service join flows, prefilled Venmo payment links, and transactional email automation. Runs on an autopilot model using scheduled GitHub Actions workflows calling protected backend API endpoints, backed by Supabase PostgreSQL and Resend email delivery.

## Machine & Host Ownership
Production runs as a serverless service on Vercel targeting `https://www.networthtennis.com` with Supabase PostgreSQL as the database backend; local development runs via `serve.py` on port 7654; `homelab.yaml` records runtime host as `unknown` (unit `networth`, lifecycle `development`). No additional multi-host server deployment evidence is declared.

## What is actually built

### 1. API Services & Backend Logic (`api/`, `lib/`)
- **`api/matching.py`**: Adaptive monthly pairing engine. Pairs players based on ranking, availability, and court preferences while checking the `matches` table to prevent repeat matchups. Supports `dry_run` mode for human preview and `clear_period` for admin rematches.
- **`api/email.py` / `api/email_delivery.py` / `api/email_policy.py`**: Transactional and scheduled email engine interfacing with Resend. Enforces safe modes via `EMAIL_DELIVERY_MODE` (`disabled`, `dry_run`, `live`) and `PUBLIC_TRANSACTIONAL_EMAILS` (`enabled`). Handles message claiming, idempotency key verification, and status tracking in `email_delivery_log`.
- **`api/auth.py`**: Server-side authentication and session token management validating tokens against the `session_tokens` table.
- **`api/matches.py`**: Match score reporting, match history retrieval, and ladder ranking calculations based on games won.
- **`api/join.py`**: Player onboarding endpoint processing signups and constructing prefilled Venmo payment link URLs.
- **`api/admin.py`**: Protected administrative endpoints for ladder management, manual match overrides, and pairing reconciliation.
- **`lib/config.py`**: Central environment configuration reading `ADMIN_EMAIL`, `CRON_SECRET`, `SITE_URL`, `SMTP_PASSWORD`, `SUPABASE_ANON_KEY`, and `SUPABASE_URL`.

### 2. Database Schema & Security (`supabase-final-setup.sql`, `migrations/`)
- **Core Domain Tables**: `league_settings` (configurable rules/match frequencies), `players` (ranks, skill levels, contact info), `player_court_preferences`, `player_availability`, `matches` (scores, winner refs, status), `match_feedback`.
- **Authentication & RLS**: `session_tokens` (table with 30-day default TTL tied to player records). Direct anonymous access to tables is revoked via RLS policies; backend service role keys bypass RLS. Views (`blocked_pairs`, `player_match_compatibility`) set `security_invoker = true`.
- **Reliability & Audit Ledgers**:
  - `automation_runs`: Tracks scheduled job execution (`action`, `period_label`, status states: `running`, `succeeded`, `failed_terminal`, `preflight_failed`, `postcheck_failed`, `repairing`, `repaired`, `summary_json`, `error_json`).
  - `email_delivery_log` / `email_log`: Canonical email delivery ledger recording batch idempotency keys, delivery outcomes (`accepted`, `failed`, `unknown`), and provider message IDs.

### 3. Frontend & Static Assets (`public/`, `content/`)
- **Web Pages**: Static HTML interface in `public/` including `index.html` (ladder & leaderboard), `join.html` (signup & Venmo payment options), `admin.html` (admin controls), `dashboard.html` (player match view), and `fallback.html`.
- **Structured Content**: JSON configurations in `content/` and `public/`: `courts.json` (East Side LA court specs), `emails.json` (notification templates), `rules.json` (league ladder rules), `how-it-works.json`, `landing.json`, `ui.json`, `league.json`.

### 4. Local Runtime & Test Infrastructure (`serve.py`, `tests/`)
- **`serve.py`**: Lightweight local dev server (`http.server` / `socketserver`) serving static files from `public/` on port 7654 with sample player fixture data (`SAMPLE_PLAYERS`).
- **Test Suite (`tests/`)**: Pytest coverage for API endpoints (`test_api.py`), email safety/delivery constraints (`test_email_safety.py`, `test_email_delivery.py`), pairing algorithm (`test_matching.py`, `test_pairings.py`), Venmo link generation (`test_join_payment_links.py`), and dependencies.

### 5. Scheduled Autopilot & Workflows (`.github/workflows/`)
- **`biweekly-emails.yml`**: Scheduled cron workflow executing availability reminders (27th), final reminders (last day of month), pairing generation + match emails (1st @ 9am PT), pairing health checks (1st @ 12pm PT), daily missing-pairing watchdogs, and mid-month pending match reminders (15th). Requires `Authorization: Bearer CRON_SECRET`.
- **`daily-health-check.yml`**: Read-only health check validating canonical host endpoints and public privacy boundaries.

## Canonical entry points
- `AGENTS.md`: Operating contract, task routing rules, worker dispatch protocol, and pairing reliability requirements.
- `api/matching.py`: Monthly opponent assignment and adaptive pairing generation logic.
- `api/email_delivery.py`: Email delivery pipeline and Resend provider integration.
- `serve.py`: Local development server runner (`python3 serve.py`).
- `public/index.html`: Web application user interface entry point.
- `supabase-final-setup.sql` & `migrations/*.sql`: Database schema definition and migration history.
- `.github/workflows/biweekly-emails.yml`: Production automation and cron scheduling workflow.
