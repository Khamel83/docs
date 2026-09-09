# LLM-OVERVIEW — maya-signing-p12-fix
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
Maya is a personal-context service and orchestration control plane. It accepts captures (text, binary files, URLs, messages), stores durable canonical records, extracts searchable text and entities, mirrors personal signals, and presents context through HTTP APIs and the Maya Studio web dashboard. It acts as an immutable domain foundation for work items, execution runs, budgets, approvals, transactional outbox delivery, and service projections.

In this branch (`maya-signing-p12-fix`), Maya includes a durable, locally signed macOS iMessage Bridge application (`dist/Maya Message Bridge.app`) packaged with PyInstaller and signed via macOS Security framework PKCS#12 (`.p12`) imports into a custom file keychain (`~/Library/Keychains/maya-signing.keychain-db`).

## Machine & Host Ownership
No multi-host information is available from the status probe output or AGENTS.md. Code configurations define local macOS execution (launchd daemons for the main FastAPI server on port 8200, iMessage bridge on port 8766, and WhatsApp daemon on port 9877), Cloudflare Worker edge drop ingress (`worker/wrangler.toml`), and homelab SOPS/age secret recovery storage paths (`services/imessage_bridge/signing.py`).

## What is actually built

- **Core FastAPI Application & Data Layer (`app/main.py`, `app/config.py`, `app/database.py`, `app/auth.py`)**:
  - Python 3.12 FastAPI service running on port `8200` by default, configured via `.env` (`app/config.py`).
  - API authentication (`app/auth.py`) enforcing `Authorization: Bearer <MAYA_INGEST_TOKEN>` (returns 503 in production if token is unset).
  - Async SQLite database (`app/database.py`) defaulting to `sqlite+aiosqlite:///data/maya.db?timeout=30` with WAL mode, foreign keys enabled, busy timeout, and NullPool (supports `postgresql+asyncpg` configuration). Schema evolution is governed by Alembic (`alembic/`).
  - Ingestion pipeline (`app/ingest_core.py`, `app/api/ingest.py`, `app/api/drop.py`, `app/api/files.py`): universal drop endpoint (`POST /ingest/drop`), base64 file upload (`POST /ingest/file`), file ledger with SQLite FTS5 search, and asynchronous background entity extraction via Ollama (`app/jobs/entity_extract.py`, `app/entity_extraction.py`).

- **Durable Signed iMessage Bridge & Security Provisioning (`services/imessage_bridge/`)**:
  - Standalone HTTP bridge server (`server.py`) listening on `127.0.0.1:8766`. Exposes `/health`, `/messages`, and `/chats` backed by macOS `~/Library/Messages/chat.db` (using Apple Epoch timestamps) with `X-Maya-Bridge-Key` file authentication.
  - Local code-signing identity provisioner (`signing.py`, `keychain_helper.swift`): generates root ("Maya Local Code Signing Root 2026") and leaf ("Maya Local Code Signing: iMessage Bridge") certificates inside dedicated file keychain `~/Library/Keychains/maya-signing.keychain-db`.
  - Security framework (`SecPKCS12Import`) integration with parameter preservation and key partition list setup (`set-key-partition-list -S apple-tool:,apple:`).
  - Encrypted recovery archive pipeline: packs root/leaf `.p12` files, certificates, designated requirements, and `recovery.env` into an age/SOPS-encrypted archive (`maya-imessage-signing-recovery.tar.age`) targeting homelab secrets.
  - Packaging and validation (`packaging.py`, `MayaMessageBridge.spec`): freezes app bundle (`dist/Maya Message Bridge.app`), signs app, verifies codesign requirements (`designated => certificate root = H"<sha1>" and identifier "com.khamel.maya.imessage-bridge"`), checks runtime entitlements, and installs LaunchAgent `com.khamel.maya.imessage-bridge.plist`.

- **Source Connectors & Signal Sync (`app/connectors/`, `app/sources/`, `app/api/`)**:
  - iMessage Bridge connector (`app/connectors/imessage_bridge.py`, `app/api/signals.py`).
  - WhatsApp Daemon connector (`app/connectors/whatsapp_daemon.py`).
  - Google Calendar sync (`app/connectors/google_calendar.py`, `app/sources/calendar_sync.py`).
  - Gmail sync (`app/connectors/gmail.py`, `scripts/gmail_backfill.py`) with Google OAuth re-authentication helper (`scripts/google_reauth.py`).
  - Slack webhooks and message registry (`app/connectors/slack.py`, `app/api/slack.py`).
  - GitHub webhook ingestion (`app/api/github_webhook.py`).
  - Granola meeting archive parser (`app/connectors/granola_archive.py`).
  - Contacts sync (`app/sources/contacts.py`, `app/sources/apple_contacts.py`, `scripts/apple_contacts_reader.swift`) matching sender identities against `contacts.yaml` priority rules.
  - Penny topic ingestion (`app/integrations/penny/`).

- **Orchestration Control Plane (`app/orchestration/`, `app/api/orchestration_*.py`)**:
  - Domain models and repositories for captures (`app/orchestration/models/capture.py`, `app/orchestration/repositories/captures.py`).
  - Work item contract and lifecycle management (`app/orchestration/models/work.py`, `app/orchestration/services/lifecycle.py`, `app/orchestration/services/decomposition.py`).
  - Execution coordination, run models, and Git outcome verification (`app/orchestration/models/execution.py`, `app/execution/`).
  - Transactional outbox delivery (`app/orchestration/services/outbox.py`), validation runner (`app/orchestration/services/validation_runner.py`), and projections (`app/orchestration/models/projections.py`).
  - Sandcastle / OMP TypeScript agent execution provider adapter (`execution/sandcastle/`).

- **Fleet Context Engine, Memory & Studio (`app/api/`)**:
  - Fleet Context Engine (`app/api/fleet.py`): assembles repository handoff, agent specs, and stack context layers for authenticated consumers.
  - Active memory endpoint (`app/api/memory.py`): `GET /memory/active` combining recent files, Slack, WhatsApp, Clio status, and fleet drift.
  - Studio UI & REST API (`app/api/studio.py`, `app/api/studio_api.py`): authenticated interface for inbox, drop ledger, routing rules, approvals, rejections, and reroutes.
  - Rules Engine (`app/rules_engine.py`): classifies captured drops against persistent rules, generating Clio work packets and audit events.

- **Auxiliary Daemons & Infrastructure**:
  - **WhatsApp Daemon (`whatsapp-daemon/`)**: Express.js server (`index.js`) on port 9877 using `@whiskeysockets/baileys`. Features an in-memory ring buffer (up to 2000 messages), JID stripping (`lib.js`), and relative time window parsing (`parseSince`). Deployed via `whatsapp-daemon/deploy/com.whatsapp-daemon.plist`.
  - **Cloudflare Edge Worker (`worker/`)**: Edge ingress proxy (`worker/src/index.js`, `wrangler.toml`) forwarding public drops to Maya's main API.
  - **Launchd Services (`deploy/`)**: `com.maya.server.plist` (FastAPI main server) and `com.maya.drop-daemon.plist` (local directory drop ingest daemon).

## Canonical entry points
- **FastAPI Main Service**: `uvicorn app.main:app --port 8200` or `scripts/start-mac.sh`
- **iMessage Bridge HTTP Server**: `python3 -m services.imessage_bridge.server` or `/Applications/Maya Message Bridge.app/Contents/MacOS/Maya Message Bridge`
- **iMessage Bridge Provisioning & Packaging**:
  - Provision signing identity: `python3 -m services.imessage_bridge.signing`
  - Build, sign & install bridge app: `python3 -m services.imessage_bridge.packaging`
- **WhatsApp Daemon**: `npm start` inside `whatsapp-daemon/` (Express server on port 9877)
- **Cloudflare Edge Worker**: `npm run dev` or `npm run deploy` inside `worker/`
- **Database Migrations**: `alembic upgrade head`
- **Test Suite**: `pytest` (executes test suite across `tests/`)
- **Launchd Service Management**:
  - Main FastAPI Server: `launchctl load deploy/com.maya.server.plist`
  - Drop Daemon: `launchctl load deploy/com.maya.drop-daemon.plist`
  - iMessage Bridge: `launchctl load services/imessage_bridge/com.khamel.maya.imessage-bridge.plist`
  - WhatsApp Daemon: `launchctl load whatsapp-daemon/deploy/com.whatsapp-daemon.plist`
