# LLM-OVERVIEW — Podemos
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
Podemos (implementation directory `podclean/`) is an automated podcast ingestion, ad detection, audio processing, and feed distribution service written in Python. It polls RSS feeds or imports OPML subscriptions, identifies ad segments via chapter parsing and audio analysis, generates cut plans to excise ads, and serves authenticated, ad-free podcast feeds alongside a management web dashboard.

## Machine & Host Ownership
No multi-host allocation information is available in the disabled status probe or `homelab.yaml` config (production host is listed as `unknown`, managed by `unknown`, systemd unit `podemos`).

## What is actually built
- **Core Orchestration & Scheduling (`podclean/src/main.py`)**: Daemon and CLI entry point configuring logging, initializing SQLite/database storage (`src.store.db`), loading runtime configuration (`src.config.config_loader.load_app_config`), running background job schedules (`BackgroundScheduler` via `apscheduler.schedulers.background`) for feed polling, episode processing, and retention cleanup, and serving the FastAPI web application via `uvicorn`.
- **Feed Ingestion (`podclean/src/ingest/`)**:
  - `rss_poll.py`: Automated polling of external podcast RSS feeds (`poll_feed`).
  - `opml_import.py`: Batch subscription ingestion from OPML files (`import_opml`).
  - Automated backlog prioritization and historical feed processing strategies.
- **Ad & Chapter Detection (`podclean/src/detect/`)**:
  - `chapters.py`: Parses JSON-formatted chapter metadata (`load_chapters_from_json`, `load_chapters_from_file`) and evaluates chapter titles against ad matching rules (`matches_ad_chapter`).
  - Detection algorithms verified via `tests/test_detect_fast.py` and `tests/test_cut_plan.py`.
- **Episode Processing Pipeline (`podclean/src/processor/`)**:
  - `episode_processor.py`: Decouples heavy full speech-to-text transcription (`perform_full_transcription`) from ad-free audio assembly (`process_episode`) for rapid delivery.
  - Error resilience: Features an episode processing retry loop bounded by `MAX_PROCESSING_RETRIES`.
- **Web Frontend & API Server (`podclean/src/serve/`)**:
  - `api.py`: FastAPI server (`api_app`) serving an administrative web dashboard for episode listings, show settings, tier management, and feed configuration.
  - Exposes `/health` monitoring endpoint and serves password/token-authenticated podcast feeds (`feat(auth)`).
- **Data Store (`podclean/src/store/`)**:
  - `db.py`: Database engine and session lifecycle functions (`init_db`, `get_session`).
  - `models.py`: ORM models including `Episode` for tracking processing status, audio cuts, and transcription state.
- **Retention & Contract Governance**:
  - Episode retention policy integrated into `BackgroundScheduler` for periodic purging of expired media files.
  - `homelab.yaml`: Project contract defined under schema `homelab.project/v1` (`id: podemos`, lifecycle `development`, kind `service`, owner `homelab`, monitoring state `standby`, systemd unit `podemos`, event sink `homelab-api:/observations`).
- **Test Suite (`podclean/tests/`)**:
  - `test_detect_fast.py`: Fast ad detection testing.
  - `test_feed.py`: RSS feed ingestion and item parsing tests.
  - `test_cut_plan.py`: Audio cutting and segment plan verification.

## Canonical entry points
- `podclean/src/main.py`: Main executable script initializing the database, scheduling background jobs (`scheduled_job_part`), and running `uvicorn`.
- `podclean/src/serve/api.py`: FastAPI application (`api_app`) hosting web management UI, feed output, and `/health`.
- `install.sh`: System setup shell script.
- `homelab.yaml`: Homelab project contract specifying service metadata and systemd unit (`podemos`).
