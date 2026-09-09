# LLM-OVERVIEW — divorce-home560-corpus-wt
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
Divorce discovery, litigation state tracking, financial modeling, communications ingestion database system, and legal exhibit package generator for *Omar Zoheri v. Meghan Logue*.

- **Authority Hierarchy**: Code and `AGENTS.md` / `CLAUDE.md` define system authority. `LITIGATION/CASE_STATUS.md` is the manual source of truth for case facts, financial numbers, and legal deadlines, superseding all generated state files. `OPERATIONS/EAGLE_STATE.md` (and derived `STATUS.md`) is a compiled read surface generated via `./eagle` or `make eagle-state`.
- **Safety Rules**: Files under `EVIDENCE/`, `SETTLEMENT/`, `LITIGATION/`, and `NEGOTIATION/` are read-only source-of-truth litigation records. Financial figures and legal deadlines must not be altered without explicit user authorization.
- **Legal Authority Verification**: Before asserting California law or drafting legally operative statements, agents must run `./eagle --legal-question "<question>"`. Only `SUPPORTED` responses within their stated scope may be cited; `UNSUPPORTED` responses halt assertions until controlling legal authority is added. Secondary leads or compiled state files are never legal authority.

## Machine & Host Ownership
- **Primary Host**: Local workspace workstation. `homelab.yaml` configures project ID `divorce`, lifecycle `development`, monitoring state `standby`, and production host `unknown`.
- **Remote Execution Runner**: Configured as `homelab` (remote repo path `/home/khamel83/github/divorce`), accessible via `Makefile` SSH targets (`make eagle-runner-check`, `make eagle-runner-state`, `make eagle-runner-run`).
- **Stateless VM Sync**: Daily state and `.eagle_cache.json` are committed to `origin/main` via `eagle_daily_commit.sh` for stateless VM access.
- **Status Probe**: Disabled (`JANITOR_RUN_STATUS_PROBE=1` to activate).

## What is actually built
- **Communications & Evidence Database (`divorce_discovery.db`)**:
  - SQLite database tracking 187,058+ to 195,796+ communications spanning 2014–2025 across multiple channels: iMessage (113,662+), SMS/Text (71,757+), Email (1,592+), Claude AI strategy conversations (47), Telegram, and daycare messaging (`comms_daycare.md`, `comms_emails.md`, `comms_telegram.md`).
  - Core tables: `communications` (multi-channel message records with timestamps), `legal_strategy_events` (145+ catalogued litigation events), `file_manifest` (26+ tracked source files).
  - Schema defined in `schema.sql` and extended via `TOOLS/add_time_views.sql`.
- **Ingestion & Data Pipelines**:
  - Ingestor modules (`ingest_texts.py`, `ingest_emails.py`, `ingest_claude.py`, `ingest_chats.py`) parse raw data from `inputs/` and `SOURCE_DOCS/`.
  - Shell orchestrators (`run_full_ingestion.sh`, `start_web_platform.sh`, `TOOLS/RUN_EVERYTHING.sh`).
  - External feeds & synchronization: Clio integration (`eagle-clio-sync-hardening`), Gmail sync (`OPERATIONS/GMAIL_SYNC_RELIABILITY_SPEC.md`, `TOOLS/GMAIL_CHECK_README.md`, background IMAP sync), LA Court docket polling (`OPERATIONS/DOCKET_SYNC_RELIABILITY_SPEC.md`, `test_poll_lacourt.py`).
- **Eagle State Compiler & Legal Intelligence Engine**:
  - Root CLI `./eagle`: Entry point for state compilation and authority validation (`./eagle --legal-question`).
  - State compilation: `TOOLS/generate_eagle_state.py` compiles `OPERATIONS/EAGLE_STATE.md` and `STATUS.md`; `TOOLS/eagle_daily` compiles `OPERATIONS/EAGLE_PARALEGAL.md`.
  - Event ledger: `OPERATIONS/EAGLE_EVENTS.jsonl` records operational events.
  - California Family Code reference corpus under `REFERENCE/CALIFORNIA_FAMILY_CODE.md` and `REFERENCE/LEGAL_CORPUS/`.
- **Discovery Response & Redaction Pipeline (`LITIGATION/03_DISCOVERY/`)**:
  - Audited manifest processing: `TOOLS/build_set1_audited_manifest_v3.py` processes vision page review manifests (`SET1_VISION_PAGE_REVIEW_V3.jsonl`), generating `SET1_VISION_DOCUMENT_SUMMARY_V3.json`, `SET1_VISION_EXCEPTION_LOG_V3.md`, and `SET1_VISION_RUN_MANIFEST_V3.json`.
  - Redaction candidate classifier: `TOOLS/classify_set1_redaction_candidates.py` evaluates exact candidate identifiers (SSN, TIN, recorded instruments) generating `SET1_EXACT_REDACTION_PLAN_V3.csv`.
  - PDF exhibit production: `TOOLS/exhibit_generation_engine.py` builds legal PDF exhibit packages using ReportLab, Matplotlib, and Pandas.
  - Bates stamping: `TOOLS/pdf_stamp.py` burns pixel-space Bates stamps onto document lower-right margins using Pillow (`ImageDraw`, `ImageFont`).
- **Financial & Settlement Subsystems (`SETTLEMENT/`, `EVIDENCE/`, `NEGOTIATION/`)**:
  - Financial separation anchor date: May 8, 2025 (and March 1, 2025 separation date for communications filtering).
  - Analysis toolkits: `TOOLS/FINAL_ANALYSIS_TOOLKIT.py`, `TOOLS/analyze_financial_documents.py`, `TOOLS/analyze_divorce_document_handoff.mjs`.
  - Financial ledgers: Betterment transaction extractions (`BETTERMENT_EXTRACT_BATCH1.md` through `BATCH3`), Fidelity extractions (`EXTRACTED_FIDELITY.md`), Zakat 2026 analysis (`ZAKAT_2026.md`), retirement fact and gap ledger, WROS share calculation formulas (`cf8ab88b`), and house valuation tracking (`aacd8ca2`).
  - Negotiation & MSA gates: MSA v6 legal review gates (`23dc46d3`), negotiation convergence controls (`020d4527`), mutual support waiver decisions (`7d5cdc76`), and child-expense account architecture (`7a2f2cab`).
- **Parenting & Agent Subsystem (`MARCUS/`)**:
  - Dedicated agent module featuring `AGENT_NARRATIVE.md`, `PARENTING_PLAN_RULES.md`, `app/`, `docs/`, `doctrine/`, database migrations, and OAuth exchange scripts (`gen_oauth_url.py`, `oauth_exchange.py`).
- **Testing & Verification Suite (`tests/`)**:
  - Automated tests covering legal corpus filter limits (`test_corpus_filter.py`), Gmail daily checks, IMAP sync, LA Court docket polling, Maya signal sync failure handling, Set 1 fact source ledgers, and deadline calculations (`test_eagle_deadlines.py`).

## Canonical entry points
- **Operational CLI & State Compilation**:
  - `./eagle`: Primary CLI wrapper for state compilation and legal question validation (`./eagle --legal-question "<query>"`).
  - `make eagle-run`: Processes recent events and regenerates local state.
  - `make eagle-state`: Runs `python3.12 TOOLS/generate_eagle_state.py` to regenerate `OPERATIONS/EAGLE_STATE.md`.
  - `make eagle-paralegal`: Runs `python3.12 TOOLS/eagle_daily` to regenerate `OPERATIONS/EAGLE_PARALEGAL.md`.
  - `make eagle-validate`: Checks file requirements and feed freshness.
  - `make eagle-events`: Displays recent ledger entries from `OPERATIONS/EAGLE_EVENTS.jsonl`.
- **Legal Corpus Verification**:
  - `make eagle-legal-status`: Shows local legal corpus status.
  - `make eagle-legal-build`: Rebuilds legal corpus index.
  - `make eagle-legal-verify`: Performs fail-closed legal corpus verification.
- **Remote Runner Control**:
  - `make eagle-runner-check`: SSH check of remote runner git and cron status.
  - `make eagle-runner-state`: SSH command to regenerate state on runner `homelab`.
  - `make eagle-runner-run`: SSH command to execute full daily Eagle pipeline on runner `homelab`.
- **Data Ingestion & Pipeline Entry Points**:
  - Ingestion scripts: `python3 ingest_texts.py`, `python3 ingest_emails.py`, `python3 ingest_claude.py`, `python3 ingest_chats.py`.
  - Pipeline orchestrators: `./run_full_ingestion.sh`, `./start_web_platform.sh`, `TOOLS/RUN_EVERYTHING.sh`.
- **Discovery Production & Redaction Tools**:
  - `python3 TOOLS/build_set1_audited_manifest_v3.py`: Generates audited Set 1 discovery review manifest (V3).
  - `python3 TOOLS/classify_set1_redaction_candidates.py`: Classifies exact Set 1 redactions.
  - `python3 TOOLS/exhibit_generation_engine.py`: Builds PDF exhibit packages.
- **Litigation & Case Authority Files**:
  - `LITIGATION/CASE_STATUS.md`: Manual source of truth for case facts, deadlines, and legal strategy.
  - `OPERATIONS/EAGLE_STATE.md`: Compiled operational briefing.
