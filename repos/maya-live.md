# LLM-OVERVIEW — maya-live
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is

Maya is a personal-context memory service and durable control plane for human captures and agent workflows. It ingests content from human drops (Slack `#maya`, macOS/iOS Share actions, REST APIs) and external connectors (WhatsApp, Gmail, iMessage, Calendar, GitHub, Granola, Penny, DataLogs), stores durable records in SQLite/PostgreSQL, extracts searchable text and entities, and presents context through HTTP REST, SQLite FTS5 search, and MCP contracts.

Maya also manages provider-neutral agent execution. Top-level requests in Slack (`#maya`) generate isolated Git worktrees, execute through provider runtimes (Sandcastle/OMP or the Python subprocess runtime), run repository validation plans, push `codex/` branches, open pull requests, and conduct automated code reviews. Maya enforces a strict human-in-the-loop requirement: it never auto-merges and never deploys. Maya Studio serves as an authenticated exception and management console. Clio is retired and off.

## Machine & Host Ownership

No multi-host probe or inventory information is available in `AGENTS.md` or status probe output (probe disabled). Code configurations define local launchd plists (`deploy/com.maya.server.plist` for port `8200` FastAPI service, `deploy/com.maya.drop-daemon.plist` for `scripts/drop-daemon.sh`), a Node.js WhatsApp daemon (`whatsapp-daemon/index.js` on port `9877`), and a Cloudflare Worker edge ingress (`worker/wrangler.toml`).

## What is actually built

- **Core Application & HTTP APIs (`app/`)**:
  - `app/main.py`: Instantiates the FastAPI app on default port `8200` (loaded via `app/config.py`), registers API routers, initializes SQLAlchemy database connections, and conditionally runs APScheduler and backfill jobs.
  - `app/config.py`: Configuration management via `pydantic-settings` reading `.env`. Core properties include `MAYA_ENV`, `MAYA_INGEST_TOKEN`, `MAYA_DATABASE_URL` (fallback `DATABASE_URL`), `MAYA_CONTACTS_PATHS`.
  - `app/auth.py` & `app/auth_session.py`: Token validation via `require_maya_token` expecting `Authorization: Bearer <token>`. Unconfigured tokens yield HTTP 503 in production and issue warnings in dev.
  - `app/ingest_core.py` & `app/api/ingest.py`: Core ingestion pipeline. `POST /ingest/drop` processes text, JSON, URL, and binary drops; accepts duplicate URL replays (`99159ac`) and verifies durable Atlas acknowledgements (`383648b`). `POST /ingest/file` handles base64 payloads and deduplicates Slack uploads.
  - `app/rules_engine.py`: Evaluates drops against persistent rules, storing decisions, attempts, work packets, and audit trails.
  - `app/api/studio.py`: Authenticated Studio endpoints for drop inboxes, file ledgers, routing rules, approvals, rejections, and rerouting.
  - `GET /memory/active`: Combines recent files, Slack drops, WhatsApp messages, and fleet drift status into active memory context, with independent degradation per section.

- **Persistence & Data Model (`app/database.py`, `alembic/`)**:
  - Async SQLAlchemy 2.0 engine (`sqlite+aiosqlite:///data/maya.db?timeout=30`) configured with WAL mode, foreign key enforcement, busy timeout, and `NullPool`. Supports `postgresql+asyncpg`.
  - Ledger & Entity Tables: Captures, Drops, Files (with extracted text via `pypdf`/`python-docx` and SQLite FTS5 search), Work Items, Routing Rules, Execution Runs, Leases, Budgets, Approvals, Artifacts, Outbox Deliveries, Projections, Contacts, and Persons (`app/orchestration/models/persons.py`).

- **Orchestration, Provider-Neutral Execution & Review (`app/orchestration/`, `app/execution/`, `app/reviews/`)**:
  - F3 State-Machine Decomposition (`9ec3a97`): Manages work item lifecycles, dependencies, execution leases, budgets, validation plans, outbox delivery, and service projections.
  - Provider-Neutral Execution (`app/execution/`, `execution/sandcastle/`): Resolves repositories, creates isolated Git worktrees, executes code modifications, runs allowlisted repo validation, pushes `codex/` branches, and opens GitHub pull requests.
  - `app/execution/subprocess_runtime.py`: Subprocess execution runtime frozen and pinned by `adapter_sha256` in `docs/evidence/p2/runtime-contract-acceptance.json` (attested at P2-07, re-attested in `7ee9a15`).
  - Automated Review Engine (`app/reviews/`): Runs automated code reviews on `codex/` PRs, classifies review-loop failures (`f6c71a6`), and retains review context on worker failures (`5028b75`).

- **Connectors & Background Processing (`app/connectors/`, `app/jobs/`, `app/services/`)**:
  - Entity Extraction (`app/entity_extraction.py`, `app/jobs/entity_extract.py`): Asynchronous background worker scanning files where `extract_status='ok'` and `entity_status='pending'`. Uses local Ollama JSON generation (`app/ollama_client.py`) on input truncated to 6000 chars to extract `people`, `projects`, etc.
  - Contact Loader (`app/contacts.py`): Scoped YAML contact parser (`MAYA_CONTACTS_PATHS`) that tags priority=1 on matching inbound drops for categories (`meghan`, `family`, `school`, `therapy`, `legal`, `gv_sms_tracked`).
  - Connector Integration: Calendar, Gmail (`scripts/gmail_backfill.py`, `scripts/google_reauth.py`), iMessage (`services/imessage_bridge`), WhatsApp daemon, Slack, GitHub, Granola, Penny, DataLogs.
  - Hermes Projection (`app/jobs/hermes_projection.py`, `docs/HERMES_BRIDGE_PLAN.md`): Projects execution receipts, thread updates, and lifecycle milestones back to Slack threads.

- **Auxiliary Services & External Runtimes**:
  - WhatsApp Daemon (`whatsapp-daemon/`): ES module Node.js application using Baileys (`@whiskeysockets/baileys`) and Express (port `9877`, `WHATSAPP_API_KEY`). Maintains an in-memory ring buffer (max 2000 messages) in `lib.js` and outputs QR pairing images to `qr.png`.
  - macOS Bridge Helpers (`scripts/apple_contacts_reader.swift`, `dist/Maya Message Bridge.app`): Swift binary for macOS Apple Contacts extraction and compiled app bundle.
  - Cloudflare Worker (`worker/`): Edge worker in TypeScript (`src/`, `wrangler.toml`).

- **Retired Systems**:
  - Clio: Clio is retired and off; Maya handles execution and review directly.

## Canonical entry points

- **FastAPI Application**: `app/main.py` (App factory, router binding, database setup, APScheduler initialization). Default port `8200`.
- **Ingestion Routes**: `app/ingest_core.py`, `app/api/ingest.py` (`POST /ingest/drop`, `POST /ingest/file`).
- **Studio Interface**: `app/api/studio.py`.
- **Active Memory Surface**: `GET /memory/active`.
- **Execution & Review Subsystems**: `app/execution/subprocess_runtime.py` (evidence-frozen runtime), `app/orchestration/`, `app/reviews/`.
- **WhatsApp Daemon**: `whatsapp-daemon/index.js` (Express server on port `9877`).
- **Development & Verification Commands**:
  - Environment setup: `make setup` (`uv venv --python 3.12 .venv && uv pip install -e ".[dev]"`)
  - Linter: `make lint` (`ruff check .`)
  - Type checker: `make types` (`pyright`)
  - Test runner: `make test` (`pytest -q`)
  - Full check suite: `make check` (`lint types test`)
- **Pyright Type Checker Scope (`pyproject.toml`)**: Scoped basic mode covering `app/jobs/orchestration_runtime.py`, `app/jobs/hermes_projection.py`, `app/reviews`, `app/execution`, `app/orchestration`, `app/config.py`.
- **Configuration Keys**: `.env` read via `app/config.py` (`MAYA_ENV`, `MAYA_INGEST_TOKEN`, `MAYA_DATABASE_URL`, `MAYA_CONTACTS_PATHS`, `PORT`, `WHATSAPP_API_KEY`).
- **Core Documentation**: `README.md`, `docs/ARCHITECTURE.md`, `docs/HERMES_BRIDGE_PLAN.md`, `docs/FLEET_CONTEXT_ARCHITECTURE.md`, `docs/USING_MAYA.md`.
