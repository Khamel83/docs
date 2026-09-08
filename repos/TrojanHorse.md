# LLM-OVERVIEW — TrojanHorse
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
TrojanHorse is a private, local-first work-evidence corpus and vault processing engine (`trojanhorse-work-corpus` v2.0.0 / `TrojanHorse` v0.2.0). The system ingests, normalizes, processes, and structures local work evidence—including `.eml` email files, MacWhisper and Wispr meeting transcripts, HyprNote exports, Granola REST API captures, Zoom logs, and local markdown notes—into a deterministic local work evidence archive (`work-corpus/corpus/`). It provides vector and SQLite-backed RAG Q&A, YAML frontmatter metadata classification (`NoteMeta`), meeting transcript synthesis (`MeetingSynthesizer`), and a FastAPI REST interface without relying on cloud processing or direct mailbox scraping.

Retired / Legacy Context: Atlas integration (`atlas_client.py`, `docs/LEGACY_ATLAS.md`), direct cloud mailbox/Outlook sync, and MCP UUID shadow feeds are retired historical components. All active evidence processing is strictly local and file-based.

## Machine & Host Ownership
- **`macmini` (`macmini-ts`)**: Primary production host declared in `homelab.yaml`. Runs the user-launchd unit `com.khamel83.work-corpus-granola-delta` for automated Granola REST API delta collection and local MacWhisper transcription.
- **`oci-ts`**: Secondary execution machine defined in the `AGENTS.md` SSH dispatch protocol for offloading `codex` worker tasks (`ssh oci-ts ...`).
- **Homelab Argus Node (`100.112.130.100:8270`)**: Centralized web search HTTP endpoint (`/api/search`) and MCP tool provider (`mcp__argus__search_web`).

## What is actually built
The repository consists of two core Python packages (`work-corpus` and `TrojanHorse`), inventory tracking, maintenance tools, and system service definitions:

- **Local Work Corpus Engine (`work-corpus/` package, v2.0.0)**
  - Configured via `pyproject.toml` as `trojanhorse-work-corpus` (source directory: `work-corpus/src/`).
  - Configuration files: `work-corpus/config.json`, `config.local.json`, `entity_aliases.example.json`.
  - Ingests raw local sources into a deterministic corpus structure (`work-corpus/corpus/`), normalizing `.eml` files locally while preserving raw source captures.
  - Maintains Granola notes via a bounded REST API delta runner, saving responses as append-only raw local files.
  - Document parsing capabilities enabled via optional dependencies (`pypdf>=5.0`, `python-docx>=1.1`, `python-pptx>=1.0`, `openpyxl>=3.1`).

- **Vault Processing & RAG Engine (`TrojanHorse/` package, v0.2.0)**
  - `TrojanHorse/config.py`: `Config` dataclass loaded via `Config.from_env()`, managing workspace directories (`vault_root`, `capture_dirs`, `processed_root`, `state_dir`, `transcripts_raw_dir`, `meetings_synthesized_dir`) and OpenRouter/embedding credentials.
  - `TrojanHorse/models.py`: `NoteMeta` dataclass parsing YAML frontmatter. Classifies notes by source (`hyprnote`, `legacy`, `unknown`), raw type (`email_dump`, `slack_dump`, `voice_note`, `meeting_transcript`, `other`), class type (`work`, `personal`), category (`email`, `slack`, `meeting`, `idea`, `task`, `log`, `other`), project, timestamps, and tags.
  - `TrojanHorse/processor.py`: Monitors capture directories, parses incoming notes, interacts with `llm_client.py` (OpenRouter API) for note classification, and outputs structured Markdown files.
  - `TrojanHorse/rag.py` & `index_db.py`: Vector search and RAG engine (`RAGIndex`, `rebuild_index`, `query`) backed by a SQLite index (`IndexDB`) stored in `state_dir`.
  - `TrojanHorse/meeting_synthesizer.py`: `MeetingSynthesizer` processing raw transcripts in `transcripts_raw_dir` with customizable templates to generate structured output in `meetings_synthesized_dir`.
  - `TrojanHorse/api_server.py`: FastAPI application (`api_app`) exposing REST endpoints for RAG Q&A, note processing, and index status. Manages lifecycle state (`config`, `index_db`, `rag_index`).
  - `TrojanHorse/cli.py`: Typer CLI wrapping processing operations, index rebuilding, RAG queries, meeting synthesis, and uvicorn API server startup.

- **Data Inventory & Ingestion Framework (`01_INVENTORY/`, `data/`, `Omar_Work_Corpus_Bootstrap_v1/`)**
  - Source tracking: `01_INVENTORY/source_manifest.csv`, `coverage_and_gaps.md`, `source_access_matrix.md`, `ingestion_plan.md`.
  - Local raw evidence data store: `data/Zoom/`, `data/notes/`, `data/mcp/`.
  - Bootstrap command wrappers: `Omar_Work_Corpus_Bootstrap_v1` containing update commands (`RUN_DAILY_LOCAL_UPDATE.command`, `OPEN_WORK_CORPUS_REPORT.command`) and legacy Outlook probes (`DISABLE_LEGACY_OUTLOOK_DAILY.command`).

- **Bridge & Maintenance Services (`bridge/`, `scripts/`, `systemd/`, `_quarantine/`)**
  - Service manifests: `scripts/com.khamel83.trojanhorse.plist` and `homelab.yaml` unit `com.khamel83.work-corpus-granola-delta`.
  - Maintenance scripts: `scripts/start_workday.sh`, `scripts/verify_setup.sh`, `scripts/oneshot-build`.
  - Legacy bridge components (`bridge/bridge_service.py`, `bridge/vacuum.py`) and deprecated systemd files quarantined in `_quarantine/`.

- **Retired Modules (`TrojanHorse/atlas_client.py`, `docs/LEGACY_ATLAS.md`)**
  - `atlas_client.py`: `AtlasClient` implementation for note promotion to external Atlas API (`http://localhost:7444`). Deprecated per `docs/LEGACY_ATLAS.md` following transition to local-only corpus storage.

## Canonical entry points
| Entry Point | Type | Purpose |
| --- | --- | --- |
| `work-corpus` | CLI | Installed CLI binary (`work_corpus.cli:main`) for local corpus ingestion and organization |
| `python3 -m TrojanHorse.cli` | CLI | Typer CLI (`trojanhorse`) for note processing, RAG indexing/querying, meeting synthesis, and API server execution |
| `TrojanHorse.api_server:app` | REST API | FastAPI application providing programmatic RAG Q&A and index management endpoints |
| `com.khamel83.work-corpus-granola-delta` | launchd unit | Production background service on `macmini` running Granola REST API delta collection |
| `python3 -m core.router.resolve` | CLI | Router resolution tool for ONE_SHOT task routing (`--class`, `--category`) |
| `http://100.112.130.100:8270/api/search` | REST API | Homelab Argus search HTTP endpoint |
| `pyproject.toml` | Config | Package build specification, pytest options (`where = ["work-corpus/src"]`), and optional dependencies |
| `homelab.yaml` | Config | Homelab project spec, production host mapping (`macmini`), launchd unit, and observability sink |
| `work-corpus/config.json` | Config | Core corpus storage directory and ingestion path definitions |
| `AGENTS.md` | Policy / Contract | ONE_SHOT v14 orchestration operating contract, intelligence tiers, and SSH routing table |
