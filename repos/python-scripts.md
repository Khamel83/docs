# LLM-OVERVIEW — python-scripts
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
`python-scripts` is a multi-utility repository containing personal automation scripts, email/media processing tools, and the embedded `Atlas` content ingestion framework. It houses pipelines for Gmail newsletter extraction and deduplication, Instapaper bookmark scraping and AI processing, video deduplication, Slack API integrations, story generation, and web-based ingestion management for articles, podcasts, and YouTube content.

## Machine & Host Ownership
Configured for `macmini` local execution (macOS `Darwin` environment using `/Users/macmini/Library/Mobile Documents/com~apple~CloudDocs/Code/` paths across scripts). `homelab.yaml` registers service metadata (`id: python-scripts`, owner: `homelab`) with `production.host: unknown` in `development` lifecycle state; no multi-host production deployment is configured.

## What is actually built
- **Atlas Subsystem (`Atlas/`)**:
  - `web/app.py`: Flask/Web application providing ingestion job logs and execution status for articles, podcasts, and YouTube channels.
  - `scheduler.db`: Centralized SQLite database at project root for job persistence across web UI and job schedulers.
  - Ingestion & Core Framework: Strategy-based content fetching via `ArticleFetcher` (`article_strategies.py`, `article_fetcher.py`), standard `FetchResult` model, `ContentAnalyzer`, `AtlasErrorHandler`, `PathManager`, and `MetadataManager`.
  - Deployment & Tools: `Dockerfile` container definition, `Atlas.code-workspace`, and environment management script `cleanup_atlas.sh`.
- **Instapaper Pipeline (`instapaper/`)**:
  - `extract_instapaper.py`: Export parser converting Instapaper CSV dumps to CSV, JSON, Excel (`.xlsx`), or Markdown.
  - `instapaper download new.py`: Multi-engine article fetcher combining Selenium WebDriver (with macOS Chrome binary detection), `trafilatura`, `newspaper3k`, `html2text`, and `BeautifulSoup`.
  - `instapaper122324v4.py`: Parallel article scraper (`ThreadPoolExecutor`) integrating `requests.Session` and local LLM processing via `langchain_ollama` (`OllamaLLM`).
  - `instapaper-viewer.tsx`: React/TypeScript interface for viewing processed Instapaper exports.
- **Email Processing Pipeline (`email downloads/`, `email processing/`)**:
  - `download_v2.py`: Gmail API client fetching emails tagged under `"Newsletter"` via OAuth2 (`googleapiclient.discovery`, `google-auth-oauthlib`).
  - `deduplicate-emails.py`: SHA-256 byte-hash deduplication (`get_file_hash`) and Gmail ID indexer producing `deduplication_report.json` and tracking state in `email_status.json`.
  - `html-content-deduplicator.py`: HTML text parsing (`bs4`) and fuzzy string matching (`difflib`) to locate duplicate email bodies (`html_deduplication_report.json`).
  - `email_processor_v6.py` & `add_unique_id.py`: Batch processor assigning unique identifiers and transforming raw CSV email data (`emails.csv`).
- **Standalone Automation Utilities**:
  - `video dedupe/analyzer_v6.py`: Video file similarity and deduplication analyzer.
  - `slack/slackv1.py`: Slack API integration script.
  - `story generator/story_generator2.py`: Automated story generation script.
  - `kid-friendly-ai/`: Workspace folder for child-safe AI script experiments.
- **Contract & Config**:
  - `homelab.yaml`: Project contract defined under `homelab.project/v1` schema with lifecycle `development` and `standby` monitoring state.
  - `config.env`: Local environment variable settings file.

## Canonical entry points
- `Atlas/web/app.py`: Web UI and API backend for job status and log streaming.
- `Atlas/cleanup_atlas.sh`: Shell utility for resetting Atlas runtime state.
- `email downloads/download_v2.py`: Gmail API newsletter fetch script.
- `email downloads/deduplicate-emails.py`: SHA-256 file hash deduplication CLI tool.
- `email downloads/html-content-deduplicator.py`: Text fuzzy matching deduplication CLI tool.
- `email processing/email_processor_v6.py`: Batch email CSV processor.
- `instapaper/extract_instapaper.py`: Instapaper export format converter (`extract_instapaper_articles`).
- `instapaper/instapaper download new.py`: Selenium + Trafilatura article downloader.
- `instapaper/instapaper122324v4.py`: Parallel article scraper and Ollama LLM summarizer.
- `video dedupe/analyzer_v6.py`: Video content deduplication execution script.
- `slack/slackv1.py`: Slack API interface script.
- `story generator/story_generator2.py`: Story generator script.
