# LLM-OVERVIEW — freeme
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
`freeme` is a self-hosted, local-first single-user privacy removal cockpit designed to manage personal data exposure across 27+ data brokers and people-search engines (e.g., Spokeo, Whitepages, BeenVerified, Intelius). It provides state-machine tracking of exposure statuses, Jinja2 template request generation (CCPA, GDPR, generic), email transmission with dry-run protection, Gmail IMAP link verification, California DELETE Act DROP checklist tracking, evidence integrity hashing (SHA256), local web UI views, and Playwright/headed Chrome guided opt-out automation.

## Machine & Host Ownership
Homelab metadata (`homelab.yaml`) registers `freeme` as project ID `freeme` (kind: `service`, lifecycle: `development`, owner: `homelab`), with monitoring set to `standby` and production host/managed_by marked `unknown`. Script inventory includes `scripts/macmini_optout.py` for guided browser opt-out automation on host `macmini`. No active live status probe output or external multi-host deployment topology is configured.

## What is actually built
- **Core Configuration & Database Layer (`freeme/config.py`, `freeme/db.py`, `homelab.yaml`)**:
  - `Settings` class (pydantic-settings using `FREEME_` prefix and `.env`) managing SQLite database connection (`FREEME_DATABASE_URL`), storage paths (`FREEME_STORAGE_DIR`, `FREEME_EXPORT_DIR`), SMTP configuration (`smtp_host`, `smtp_port`, `smtp_username`, `smtp_password`, `smtp_from`, `smtp_use_tls`), safety controls (`dry_run=True`, `email_send_enabled=False`), Playwright enablement (`playwright_enabled`), Gmail IMAP credentials (`gmail_imap_account`, `gmail_imap_password`), and verification email routing (`verification_email`).
  - SQLAlchemy 2.0 ORM engine factory (`freeme/db.py`) enforcing SQLite foreign key constraints (`PRAGMA foreign_keys=ON`) via connection hooks.

- **Data Models & Data Seeding (`freeme/models.py`, `freeme/schemas.py`, `freeme/seed.py`)**:
  - SQLAlchemy ORM models (`Base`, `PersonProfile`, `Broker`, `Exposure`, `RemovalRequest`, etc.) with a custom `JSONList` TypeDecorator for JSON list handling in SQLite.
  - Pydantic schemas in `freeme/schemas.py` (`BrokerOut`, `ExposureOut`, `RemovalRequestOut`) enforcing interface types.
  - Database seed module (`freeme/seed.py`) loading 27+ broker targets from `freeme/data/seed_brokers.yaml`.

- **Removal Request & Email Verification Subsystems**:
  - Jinja2 request generation engine compiling legal opt-out notices.
  - CCPA email sender engine operating under strict dry-run guards.
  - Gmail IMAP verifier for checking broker confirmation messages and verification links.

- **Automation Engine & Scripts (`scripts/macmini_optout.py`, Playwright integration)**:
  - Playwright integration engine for automated browser-based opt-out form submissions.
  - Guided script (`scripts/macmini_optout.py`) executing headed Chrome automation for manual/interactive opt-out steps on `macmini`.

- **Web Dashboard & Command Line Interface (`freeme/web/app.py`, `freeme/cli.py`)**:
  - FastAPI application (`freeme/web/app.py`) providing a 9-view local dashboard without authentication.
  - Typer CLI (`freeme.cli:app`) supporting commands including `init-db`, `seed`, `profile`, and `validate`.

- **Evidence Storage & Export Engines**:
  - SHA256 integrity-verified storage layer with path-traversal protection for archiving proof files.
  - Exporters generating Markdown and CSV reports alongside California DELETE Act DROP program tracking.

## Canonical entry points
- **CLI Executable**: `freeme` (`freeme.cli:app` defined in `pyproject.toml`)
- **Web UI Application**: `uvicorn freeme.web.app:app --host 0.0.0.0 --port 8000` (or `freeme/web/app.py`)
- **Database Initialization**: `freeme init-db`
- **Broker Database Seeding**: `freeme seed`
- **System Validation**: `freeme validate` / `./scripts/validate.sh`
- **Guided Opt-Out Script**: `python scripts/macmini_optout.py`
- **Container Service**: `docker-compose.yml` exposing port `8000` mounted to `freeme.db`, `./storage`, and `./exports`
