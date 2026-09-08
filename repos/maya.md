# LLM-OVERVIEW — maya
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
Maya is a personal intelligence core, personal data aggregator, and context assistant system. It provides unified data ingestion pipelines (Gmail, Slack, iMessage, WhatsApp, Google Calendar, Granola meetings), background collector scheduling, Personal Access Token (MPAT) management, outbox task orchestration, and multi-source provider integrations.

- **Canonical repository checkout**: `/Volumes/2TB_SSD/GitHub/maya` (development / source of truth).
- **Operational Cutover Status**: Homelab canonical core cutover (Packet H1) executed on 2026-09-07. Production core API, PostgreSQL database (104+ tables), and drop processing run on the homelab Docker host (`homelab`). Mac launchd API wrapper is retained as a read-only compatibility bridge for legacy internal consumers (`argus`, `hermes`).
- **Data Ingestion Pipeline**: External drops land at Cloudflare Worker edge -> stored in R2 / queued -> forwarded to `https://maya.khamel.com/ingest/drop` -> processed by core transaction pipeline (`app/ingest_core.py`) -> stored in PostgreSQL and outbox dispatched.
- **Verification Standard**: Code changes require strict operational validation using `make check` (`ruff`, `pyright`, `pytest`), `python3 scripts/maya-status.py`, and `.audit/HEALTH_CHECK_RUNBOOK.md`.

## Machine & Host Ownership
- **Homelab Docker Host (`ssh homelab`, `~/github/homelab` - Ubuntu 24.04)**:
  - Primary production runtime host for the canonical Maya core service (`maya-core`) post-H1 cutover (executed 2026-09-07).
  - Container stack: `maya-api` (published on host port `18200`), `maya-scheduler`, `maya-postgres` (PostgreSQL 16, 104+ public tables). Defined in `services/maya/docker-compose.yml` in `Khamel83/homelab`.
  - Data storage volumes: `/mnt/main-drive/appdata/maya/{postgres,objects,backups}`.
  - Deploy pipeline: Push `Khamel83/homelab` on GitHub → pull on homelab host → `docker compose up -d`.
  - Public ingress: `https://maya.khamel.com` routes via Cloudflare Tunnel (`khamel-tunnel` / `Homelab` ID `0267e858-7457-4947-a0ad-153610c3a3ce`) directly to `192.168.7.10:18200`.
- **Macmini Host (`macmini` - macOS environment)**:
  - Canonical development checkout: `/Volumes/2TB_SSD/GitHub/maya`.
  - Local deployed runtime source: `/Volumes/2TB_SSD/GitHub/maya-live` (historical Mac runtime).
  - Deployed releases directory: `/Volumes/2TB_SSD/GitHub/maya-releases/` (managed by external deploy controller; never edit directly).
  - Environment configuration: `/Users/macmini/Library/Application Support/Maya/runtime/maya.env` (mode `0600`).
  - Active local daemons:
    - `mac_worker` (`scripts/mac_worker_replay.py`, `com.maya.mac-worker-replay`): Resumable iMessage cache collector & replay daemon targeting core DB.
    - `imessage_bridge` (`services/imessage_bridge/`, port 8766, `com.khamel.maya.imessage-bridge.plist`): Local macOS iMessage reader service.
    - `whatsapp-daemon` (`whatsapp-daemon/`, `com.whatsapp-daemon.plist`): Baileys Node.js daemon mirroring WhatsApp group chat events.
    - `pipeline-watchdog` (`scripts/maya-pipeline-watchdog.sh`, `com.maya.pipeline-watchdog.plist`): Local process watchdog.
  - Fenced components: Mac scheduler (`com.maya.scheduler`) is unloaded to prevent dual-writer conflicts. Mac API (`com.maya.server`) runs as a read-only compatibility bridge at `192.168.7.165:8200` for `argus` (`MAYA_CAPTURE_URL`) and `hermes` (`MAYA_BASE_URL`).
- **Cloudflare Edge**:
  - Edge environment running `worker/` (`maya-drop-ingress`) Cloudflare Worker drop front door. Receives drop payloads (R2 storage + queue consumer) and forwards drops to core `https://maya.khamel.com/ingest/drop`. Deployed directly via `wrangler`.

## What is actually built
- **Core Engine & Web API (`app/`)**:
  - `app/main.py`: FastAPI lifecycle, router registration, and health endpoints (`/healthz`, `/livez`, `/readyz`).
  - `app/config.py`: Typed Pydantic configuration settings, environment loaders, and feature flags.
  - `app/database.py`: Async SQLAlchemy engine, session lifecycle, and SQLite WAL / PostgreSQL dialect pragmas.
  - `app/api/`: REST API route modules (`drop.py`, `ingest.py`, `studio_api.py`, `mac_worker.py`, `personal_access.py`, `web_assistant.py`, `slack.py`, `vikunja_webhook.py`, `github_webhook.py`).
  - `app/models/db.py`: Complete ORM schema for files, cache, fleet management, task routing, outbox, and audit logs (104+ tables).
  - `app/ingest_core.py`: Core ingestion transaction pipeline, reservation engine, and downstream dispatching.
  - `app/jobs/` & `app/scheduler.py`: Scheduled collectors (`gmail_sync.py`, `slack_ingest.py`, `imessage_sync.py`, `whatsapp_ingest.py`, `calendar_sync.py`, `vikunja_sync.py`, `session_summarize.py`) gated by feature flags.
  - `app/orchestration/`: Capture pipelines (`repositories/captures.py`), work items (`contracts/work_item.py`), policy checks, outbox promotion (`services/promotion.py`), task decomposition (`services/decomposition.py`), project onboarding (`services/project_onboarding.py`), and quarterback execution (`quarterback/`).
  - `app/personal_access/`: MPAT token issuance (`service.py`), catalog management (`catalog.py`), scope enforcement (`gmail.py`, `imessage.py`), and evidence receipts (`evidence.py`).
  - `app/assistant/`: `#maya` assistant turn contract (`service.py`), personal response generation (`personal_answer.py`), and retrieval engines (`app/assistant/retrieval.py`).
  - `app/connectors/`: Provider adapters (`gmail.py`, `whatsapp.py`, `google_calendar.py`, `granola_archive.py`, `google_auth.py`).
  - `app/integrations/`: Internal consumer adapters (`vikunja/`, `homelab/`, `baywatch_inbox.py`).
  - `app/model_gateway/`: LLM gateway runtime integration (`runtime.py`, `adapters/openai_compatible.py`).
  - `alembic/`: Database migrations for schema evolution (supports SQLite WAL and PostgreSQL 16).
- **Daemons, Edge & External Workers**:
  - `worker/`: Cloudflare Worker drop front door (`maya-drop-ingress`, TypeScript/wrangler) with R2 storage (`src/index.js`) and queue consumer (`src/r2_queue_consumer.js`).
  - `app/mac_worker/` & `scripts/mac_worker_replay.py`: Resumable iMessage cache collector and replay daemon targeting core DB.
  - `services/imessage_bridge/`: Local macOS iMessage reader service (PyInstaller package with `keychain_helper.swift` and `server.py`).
  - `whatsapp-daemon/`: Baileys Node.js daemon (`index.js`, `lib.js`) mirroring WhatsApp group chat events to Maya API.
  - `execution/sandcastle/`: Optional TypeScript redaction worker client (`app/execution/sandcastle_client.py`), not in running path.
- **Operational & Audit Infrastructure**:
  - `scripts/maya-status.py`: Primary status probe verifying running service SHA, git state, and database health.
  - `scripts/maya_acceptance.py`: Isolated H1-H8 acceptance harness and rollback-delta rehearsal (Packet G1).
  - `scripts/maya_backup_manifest.py`: Backup manifest, verification, and restore rehearsal runner (Packet F2).
  - `scripts/maya_postgres_import.py` & `scripts/maya_rollback_delta.py`: PostgreSQL import and rollback delta reconciliation tools.
  - `.audit/`: Storage topology maps (`STORAGE_TOPOLOGY.md`), audit reports (`AUDIT_REPORT.md`), and health runbooks (`HEALTH_CHECK_RUNBOOK.md`).
- **Retired / Stranded History**:
  - Apple Rails projection retired per ADR 0029 (`codex/apple-rails-projection`).
  - Dossier / Ask Maya browser retrieval (`app/dossier/`) code merged on main, but depends on optional Graphiti/FalkorDB setup; historical rows remain in DB.

## Canonical entry points
- `python3 scripts/maya-status.py`: Primary status probe for live service, git SHA, and database health.
- `make check`: Complete code validation suite (`ruff`, `pyright`, `pytest`).
- `make setup`: Fresh environment setup and tool bootstrap.
- `app/main.py`: Core FastAPI service lifecycle and REST API entry point.
- `(cd worker && wrangler deploy)`: Cloudflare Worker drop ingress deployment command.
- `docker compose -f services/maya/docker-compose.yml --profile maya up -d`: Homelab core service stack launch command (on `ssh homelab`).
- `scripts/maya_acceptance.py`: H1-H8 acceptance and verification suite runner.
- `python3 scripts/maya_postgres_import.py`: Data import script for migrating SQLite history to PostgreSQL.
- `python3 scripts/mac_worker_replay.py`: Resumable iMessage cache collector replay daemon entry point.
- `launchctl kickstart -k gui/501/com.maya.server`: Restart local Mac compatibility bridge API wrapper.
