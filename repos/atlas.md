# LLM-OVERVIEW — atlas
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
Atlas is a personal knowledge archive that ingests podcasts, web articles, and newsletters into a unified, semantically searchable repository. Ingested media is cleaned, stored in PostgreSQL, indexed for hybrid full-text search (FTS5) and vector semantic search (`sqlite-vec`), and made accessible through a REST API, a Manifest V3 Chrome extension, and an interactive Q&A subsystem (Atlas Ask).

The system operates on a fail-fast ingestion model across 13 transcript resolvers and web extraction routines. Audio ingestion features robust transcript integrity controls, including pre-ASR download failure recovery, transcript tail clamping, historical feed snapshot binding, and offline replay evidence adjudication.

## Machine & Host Ownership
No explicit multi-host deployment information is available in AGENTS.md or the disabled status probe output (`AGENTS.md` configures remote SSH worker dispatch targets `oci-ts` and `macmini-ts`).

## What is actually built
- **Data & Storage Subsystem**:
  - PostgreSQL Database: Primary single source of truth for content items, metadata, quality verification statuses, and transformation lineage.
  - FTS & Semantic Search: Derived SQLite FTS5 full-text search index combined with `sqlite-vec` embeddings for hybrid vector retrieval.
  - Storage Adapters (`modules/storage/`): Markdown file-based abstraction layer backed by PostgreSQL persistence.
- **Podcast Pipeline & Transcript Recovery**:
  - Episode Discovery & CLI (`modules/podcasts/cli.py`): RSS feed parsing, episode metadata management, and ingestion execution.
  - Transcript Resolvers (`modules/podcasts/resolvers/`): 13 provider-specific transcript extraction engines.
  - Replay & Pre-ASR Recovery: Pre-ASR download failure recovery, exact-map replay adjudication proofs, no-cut transcript tail clamping, retained replay evidence finalization, and historical snapshot binding.
  - Quality Verification (`modules/quality/`) & Lineage (`modules/lineage/`): Automated detection of truncated or garbage content (`fetch_status`) and transformation lineage tracking.
- **Article & Ingestion Pipelines**:
  - Continuous URL Processor (`scripts/continuous_url_processor.py`): Fail-fast continuous worker for web article ingestion.
  - Capture Inbox (`modules/capture/`): Asynchronous staging area for delayed item processing.
  - Chrome Extension (`extensions/chrome-save-to-atlas/`): Manifest V3 browser extension for direct page saving.
- **Search, Q&A & Web Extraction**:
  - Atlas Ask (`modules/ask/`): Semantic Q&A engine and natural language query execution layer.
  - Link Extraction Pipeline (`modules/links/`): Embedded link extraction, relevance scoring, and approval pipeline.
  - Argus Search Integration (`modules/argus_client.py`): Unified client for web search, extraction, and completeness assessment via Argus (`${ARGUS_BASE_URL:-http://127.0.0.1:8005}`).
- **Orchestration & Governance**:
  - ONE_SHOT v14 Contract (`AGENTS.md`): Defines task routing (`plan`, `research`, `implement_small`, `test_write`, `review_diff`, `doc_draft`), intelligence tiers (`glm_claude`, `codex`, `gemini_cli`, `free`), dispatch protocols, and search routing.

## Canonical entry points
| Subsystem | Location | Purpose |
| --- | --- | --- |
| REST API | `api/main.py` | FastAPI application serving ingestion, search, and admin endpoints |
| Podcast CLI | `modules/podcasts/cli.py` | Discovery, transcript fetching, and replay recovery workflows |
| Provider Resolvers | `modules/podcasts/resolvers/` | 13 transcript extraction provider implementations |
| Continuous Ingestion | `scripts/continuous_url_processor.py` | Standalone runner for continuous article ingestion |
| Atlas Ask | `modules/ask/` | Semantic search and natural language Q&A engine |
| Argus Search Client | `modules/argus_client.py` | Single integration point for web search and web extraction |
| Storage Adapter | `modules/storage/` | PostgreSQL-backed content storage implementation |
| Quality Assurer | `modules/quality/` | Truncation/garbage checks and `fetch_status` management |
| Lineage Tracker | `modules/lineage/` | Content origin and transformation history tracking |
| Link Pipeline | `modules/links/` | Link extraction, scoring, and approval workflows |
| Capture Inbox | `modules/capture/` | Staging module for asynchronous article capture |
| Chrome Extension | `extensions/chrome-save-to-atlas/` | Manifest V3 browser extension source |
| Worker Router | `python3 -m core.router.resolve` | Task classification and worker resolution CLI script (`AGENTS.md`) |
