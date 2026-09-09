# LLM-OVERVIEW — gmail-bookmark-service
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
A Python 3.9+ service (`gmail-bookmark-service` v1.0.0) that ingests Gmail messages in real-time via GCP Pub/Sub push notifications, extracts embedded URLs and attachments, deduplicates processed content, and persists extracted links as structured bookmarks in SQLite.

## Machine & Host Ownership
No multi-host setup is active; host configuration is unconfigured (`runtime.production.host: unknown`, `managed_by: unknown` in `homelab.yaml`) and managed as a single Linux systemd unit (`gmail-bookmark-service.service`).

## What is actually built
- **Service Engine (`src/gmail_bookmark_service/main.py`)**: FastAPI application using `lifespan` context management to initialize `structlog` logging, setup SQLite schema (`init_database`), configure `CORSMiddleware`, attach `RequestLogger`, and manage background reliability tasks.
- **Configuration & Environment (`src/gmail_bookmark_service/settings.py`)**: Pydantic `BaseSettings` managing OAuth2 paths (`config/gmail_credentials.json`, `data/gmail_token.json`), watch label filter (`gmail_watch_label`), GCP Project ID (`gcp_project_id`), Pub/Sub topic (`gmail-notifications`), and subscription (`gmail-push-subscription`).
- **Database Storage (`src/gmail_bookmark_service/database/`)**: Async SQLAlchemy 2.0 layer backed by `aiosqlite`. Data models (`models.py`):
  - `Bookmark`: Stores `gmail_message_id` (unique index), `gmail_thread_id`, `subject`, `sender_email`, `sender_name`, and extracted `urls` (JSON).
  - `ProcessingState`: Tracks ingestion state and sync status.
  - `FailedMessage`: Logs failed message IDs and errors for failure recovery processing.
- **API & Ingestion Endpoints (`src/gmail_bookmark_service/api/`)**:
  - `webhook_router`: Endpoint receiving GCP Pub/Sub push notification webhooks for incoming email triggers.
  - `health_router`: Service readiness/liveness health probes and telemetry `metrics`.
- **Extraction Engine & Parsing**: HTML/text parsing using BeautifulSoup4 (`bs4`, `lxml`) and `httpx` to extract embedded URLs and download attachments, using message hash calculation for duplicate suppression.
- **Reliability & Background Subsystems (`src/gmail_bookmark_service/utils/reliability.py`)**:
  - `watch_manager`: Automated background task renewing expiring Gmail API push watch subscriptions (7-day lifecycle).
  - `failure_recovery_manager`: Circuit breaker, retry handlers, and daily catch-up fallback scanner for unhandled inbox messages.
- **Integrations & Key Dependencies**: `fastapi`, `uvicorn`, `google-api-python-client`, `google-auth-oauthlib`, `google-cloud-pubsub`, `sqlalchemy`, `aiosqlite`, `pydantic`, `pydantic-settings`, `httpx`, `beautifulsoup4`, `lxml`, `cryptography`, and `structlog`.

## Canonical entry points
- **Development Entry Point**: `python3 run.py` (executes `gmail_bookmark_service.main.main()` with dev environment settings).
- **Application Module**: `python3 -m gmail_bookmark_service.main` / `uvicorn`.
- **Package Build/Install**: `pip install -e .` (Hatchling build backend).
- **Systemd Unit**: `gmail-bookmark-service.service`.
- **Test Runner**: `pytest` (configured under dev dependencies).
