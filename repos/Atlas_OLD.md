# LLM-OVERVIEW — Atlas_OLD
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
Atlas_OLD is a personal content consumption pipeline and searchable knowledge repository designed to discover, extract, process, and index podcast transcripts, web articles, newsletters, and video captions into a SQLite database (`data/atlas.db`). Originally structured as a continuous background processing engine (v1/v2 legacy stack), the system accumulates high-value transcripts for 253 priority podcasts. The repository is marked legacy (`atlas-old` contract in homelab) as architecture shifts from continuous background queue processing toward event-driven ingestion triggers (via Velja, Hyperduck, Gmail webhooks, and Apple Shortcuts). Its primary preserved asset is a 9,566-transcript knowledge base with validated extraction patterns and user show configurations.

## Machine & Host Ownership
No multi-host information is available in the status probe output or AGENTS.md.

## What is actually built

### 1. Database & Core Data Assets (`data/atlas.db`)
- **`content` Table**: Primary knowledge store containing 9,566 extracted transcripts. Schema includes `id`, `url` (UNIQUE), `title`, `content` (full transcript text, minimum 10,000 characters for verified quality), `content_type` (`podcast`, `newsletter`, `youtube`, `article`), `metadata` (JSON payload containing publication date, show notes, extracted links, and host details), timestamps (`created_at`, `updated_at`), processing state (`stage`, `processing_status`), and AI enrichment fields (`ai_summary`, `ai_tags`, `ai_socratic`, `ai_patterns`, `ai_recommendations`, `ai_classification`).
- **`episode_queue` Table**: Queue management table recording 5,168 prioritized podcast episodes across pending, found, and error states (`id`, `podcast_name`, `episode_title`, `episode_url`, `status`, `created_at`, `updated_at`).
- **Migration & Backup Layer**: Schema migration scripts (`atlas_v2_actual_migration.py`, `universal_migration.py`, `backlog_migration.py`) and backup tooling (`create_baseline_backup.py`, `atlas_manager_db_backup.py`).

### 2. Transcript Discovery & Quality Validation Engine
- **Priority Show Processing (`your_podcast_processor.py`, `your_podcast_discovery.py`)**: Targeted pipeline for 253 user-configured priority podcasts (`config/your_priority_podcasts.json`) using direct site parsers (`config/your_podcast_sources.json`). Includes source-specific DOM extractions (e.g., Acquired `rich-text-block-6` pattern generating 95K+ character extractions).
- **Quality Gatekeeper (`transcript_quality.py`, `transcript_quality_validator.py`, `transcript_judge.py`)**: Enforces a strict 10,000-character minimum length constraint to discard metadata-only web pages, show notes, and navigation noise while preserving true full-length transcripts.
- **Multi-Source Discovery Hunters**: Extraction fallback cascade (`quality_assured_transcript_hunter.py`, `google_powered_transcript_finder.py`, `tavily_transcript_finder.py`, `mass_rss_transcript_extractor.py`) leveraging RSS feeds, direct HTML scraping, Tavily AI search, and Google Search API integrations (`setup_google_search.py`).

### 3. Multi-Channel Ingestion & Event Bridges
- **Browser & Link Ingestion (`velja_integration.py`, `url_ingestion_service.py`, `universal_url_processor.py`, `ingest/link_dispatcher.py`)**: Integrates with macOS Velja browser picker and direct link dispatchers for on-demand content submission.
- **Media Download Bridge (`custom_hyperduck_bridge.py`, `hyperduck_velja_integration_research.md`)**: Ingests media files and podcast episodes using Downie integration with failure detection.
- **Email & Newsletter Ingestion (`webhook_email_bridge.py`, `email_atlas_bridge.py`, `atlas_v3_gmail.py`, `email_to_https_bridge.py`)**: Ingests real-time Gmail bookmarks and incoming newsletter emails directly into the content pipeline.
- **YouTube Caption Scraper (`youtube_caption_scraper.py`, `youtube_auth_simple.py`)**: Scrapes and parses automated captions and transcripts from YouTube video URLs.

### 4. API, Web Dashboard & MCP Subsystems (`api/`, `web/`, `search/`)
- **FastAPI REST Service (`api/main.py`, `start_api.py`, `api/search_server.py`)**: Endpoints for transcript queries, submission captures, queue control, cognitive pipeline processing, and dashboard statistics (`api/routers/`).
- **Web Interface & Real-time Monitoring (`web_interface.py`, `start_web.py`, `monitoring_service.py`, `standalone_monitoring_service.py`)**: Web management UI and WebSocket service serving live metrics, health scores, log viewers (`log_views.py`), and queue statuses.
- **Model Context Protocol Server (`mcp_server.py`)**: Exposes Atlas search and transcript knowledge base directly to LLM agents via standard MCP protocols.

### 5. Legacy Continuous Operations & Systems Management (Retired / Superseded)
- **Continuous Loop Manager (`atlas_manager.py`, `enhanced_monitor_atlas_fixed.sh`)**: Background daemon and shell monitor designed to process queue backlogs continuously. Documented as retired architecture due to CPU overhead and queue backlog bloat, favoring on-demand event-driven execution.
- **Alerting & Notifications (`ntfy_atlas_bridge.py`, `telegram_alerts.py`, `email_alerts.py`)**: Outbound notification hooks sending status updates to ntfy, Telegram, and email channels.
- **Systemd & Apple Shortcuts Packages (`systemd/`, `apple_shortcuts/`, `shortcuts_package/`)**: Service definitions and iOS/macOS Shortcut workflows (`setup_iphone.py`, `SHARE_SHEET_SETUP.md`) for mobile share-sheet triggers.

## Canonical entry points
- `python3 your_podcast_processor.py` — Run priority podcast queue extraction processor.
- `python3 your_podcast_discovery.py` — Discover transcripts for configured priority shows.
- `python3 start_api.py` / `uvicorn api.main:app` — Launch REST API for transcript ingestion and search.
- `python3 start_web.py` / `python3 web_interface.py` — Launch web management dashboard UI.
- `python3 api/search_server.py` — Start dedicated transcript search engine server.
- `python3 mcp_server.py` — Run MCP server for LLM client integration.
- `python3 webhook_email_bridge.py` — Run Gmail/newsletter webhook ingestion bridge.
- `python3 url_ingestion_service.py` / `python3 start_url_worker.py` — Process incoming URL ingestion queue.
- `python3 youtube_caption_scraper.py` — Extract captions from targeted YouTube content.
- `python3 atlas_manager.py` — Legacy continuous background processing orchestrator.
- `./enhanced_monitor_atlas_fixed.sh` — Legacy background daemon monitoring script.
