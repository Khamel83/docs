# LLM-OVERVIEW — tix
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
`tix` (also packaged under the legacy module name `ticket_sniper`) is a self-hosted, local-first personal ticket deal monitor engineered for unattended homelab operation. It monitors secondary ticketing markets (specifically SeatGeek), applies multi-tier gate filtering against aggregate pricing stats and customized section/pricing rules, collects detailed listing data via external scrapers, and dispatches real-time deal alerts via Telegram. `Ticket Sniper` and `tix` refer to the same unified system and repository.

## Machine & Host Ownership
No multi-host information is available in the repository configurations or status probe output; the application is configured to run as a single containerized service via Docker Compose on local host `127.0.0.1:8000`.

## What is actually built
- **Core Environment & Dependencies**: Python `>=3.11` application configured via `pyproject.toml` using `setuptools`. Production dependencies include `fastapi` (>=0.110.0), `uvicorn[standard]` (>=0.28.0), `sqlalchemy` (>=2.0.28), `alembic` (>=1.13.1), `httpx` (>=0.27.0), `pydantic-settings` (>=2.2.1), `apscheduler` (>=3.10.4), `jinja2` (>=3.1.3), `curl-cffi` (>=0.6.2), and `numpy` (>=1.26.4). Development requires `pytest` and `pytest-asyncio`.
- **Database & Persistence (`ticket_sniper.db`)**: SQLite database stored at `/data/tickets.sqlite3` with schema managed through Alembic migrations (`alembic/`). Schema models:
  - `Venue`: Target venue metadata and mappings.
  - `SourceEvent`: Event schedules, external IDs, and active monitoring state.
  - `EventPriceSnapshot`: Aggregate event price metrics and gate evaluation decisions.
  - `Rule` & `RuleSectionMatcher`: Alert thresholds, seating section matchers, exact quantity splits, all-in unit price limits, maximum order totals, fee confidence levels, and unique rule fingerprints (`compute_rule_fingerprint`).
  - `AlertOutbox`: Transactional notification outbox storing deduplication keys (`dedupe_key`), JSON payloads, status states, and delivery attempt logs.
- **Event Discovery & Profiles (`ticket_sniper.discovery`, `ticket_sniper.profiles`)**:
  - `SeatGeekDiscovery`: Resolves configured venues and discovers upcoming events using `SEATGEEK_CLIENT_ID`.
  - `ProfileSpec`, `VenuePreference`, `seed_profile`: Profile configuration system allowing user preferences across sports and venues.
- **Gate Evaluator & Fee Model (`ticket_sniper.gates`, `ticket_sniper.fee_model`)**:
  - `evaluate_event_gate`: Tier-1 screening function evaluating aggregate event price snapshots against a baseline margin (`GATE_MARGIN`, default 0.15), returning decisions (`pass`, `insufficient_data`) to trigger deeper listing collection.
  - Fee modeling module for calculating fee confidence and all-in ticket prices.
- **Listing Collection & Argus Integration (`ticket_sniper.argus`)**:
  - Deep listing extraction and price history tracking integrating with an external Argus API service (`ARGUS_BASE_URL`, `ARGUS_API_KEY`).
  - Evaluates individual listings against section matchers, quantity constraints, and order totals.
- **Alert & Notification Outbox (`ticket_sniper.notifications`, `ticket_sniper.alerts`)**:
  - `process_pending_outbox`: Worker process draining `AlertOutbox` entries to Telegram (`TELEGRAM_BOT_TOKEN`, `TELEGRAM_CHAT_ID`).
  - Implements duplicate suppression, re-alert tracking, error logging, and credential verification (flagging `skipped_no_credentials` when bot configuration is missing).
- **Health & Deadman Monitoring (`ticket_sniper.health.deadman`)**:
  - `run_deadman_switch_check`: Periodic deadman monitor running on a configurable interval (`DEADMAN_HOURS`, default 6h) that enqueues heartbeat alerts (`deadman:tier2:...`) into `AlertOutbox`.
- **Operator Web Dashboard (`ticket_sniper.web.app`)**:
  - FastAPI application rendering HTML views via Jinja2.
  - `GET /`: Dashboard listing active events, listing collections, gate snapshots, poll runs, outbox status, and profile preferences.
  - `POST /events/seatgeek/{source_event_id}/poll`: Endpoint for triggering immediate manual polling cycles for an event.
  - `GET /health`: Healthcheck endpoint tested by Docker container healthprobes.
- **Prototype / Demo Seed System (`ticket_sniper.demo.seed`)**:
  - Supports fixture-backed offline prototyping (`TIX_PROTOTYPE_MODE=1`) seeding test events for Dodger Stadium and Hollywood Bowl (`seed_demo_data`).

## Canonical entry points
- **Web Server & Dashboard**: `ticket_sniper.web.app:app` served via `uvicorn` inside Docker listening on `127.0.0.1:8000`.
- **Docker Compose**: `docker-compose.yml` launching service `ticket-sniper` with volume `sniper_data` mounted to `/data`.
- **Database Migrations**: `alembic upgrade head` executing via `alembic.ini` and `alembic/env.py`.
- **Background Scheduler**: APScheduler jobs orchestrating event discovery, dynamic polling, outbox processing (`process_pending_outbox`), and deadman switch checks (`run_deadman_switch_check`).
- **Test Suite**: `pytest` executed against the `tests/` directory (e.g., `test_alert_pipeline.py`, `test_deadman.py`, `test_event_polling.py`, `test_gate_evaluator.py`, `test_notifications.py`, `test_profiles.py`).
