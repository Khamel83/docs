# LLM-OVERVIEW — divorce
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
This repository is the central litigation support system, legal discovery database, financial forensics workspace, and settlement packaging engine for Omar Zoheri's pro se representation in *Zoheri v. Logue* (Case No. 26STFL02650, LASC Department ST831).

The repository covers six core functional domains:
1. **Discovery Database & Evidence Management**: Ingestion, storage, and cross-referencing of 187,000+ communications spanning 11+ years (iMessage, SMS, Email, Claude AI strategy sessions) inside a central SQLite database (`divorce_discovery.db`) and structured normalized Markdown records (`EVIDENCE/comms_emails.md`, `EVIDENCE/comms_texts.md`, `EVIDENCE/custody_calendar.md`, `EVIDENCE/incident_log.md`).
2. **Financial Forensics & Separate Property Tracing**: S&P 500 multiplier and *Ciprari* separate property growth calculations (`SETTLEMENT/SP500_METHODOLOGY.md`), $591,493.47 WROS joint account tracing, post-separation retirement pool allocations ($869,505.03 pool), house equity schedules, and integration with the preserved float-omar forensic runtime database (`trace11.db`).
3. **Operational Briefing Pipeline ("Eagle")**: Implementation of the compiled-state pattern (`OPERATIONS/compiled_state_pattern.md`) that compiles source health, deadlines, comms logs, tasks, and events into a single operational snapshot (`OPERATIONS/EAGLE_STATE.md`) and brief (`STATUS.md`).
4. **Legal Authority Corpus & Citation Verification**: Maintenance of a hash-anchored California legal corpus (`TOOLS/legal_corpus/`, `REFERENCE/LEGAL_CORPUS/`) enforcing primary legal authority checking (`./eagle --legal-question`, `./eagle --audit-draft`) to prevent unverified legal claims.
5. **Settlement Packaging & Document Production Engine**: Drafting and rendering of the operative Marital Settlement Agreement (MSA v7.9, `NEGOTIATION/DRAFTS/MSA_v7_9_COUNSEL_REVIEW.md`), term sheets, DOCX/PDF proposal packages with redlines, Bates-stamped discovery productions (Set 1), and proof-of-service instruments.
6. **Pro Se Communication & Voice Gate**: Tactical and legal review pipeline translating outbound legal artifacts into Omar's pro se voice (`OPERATIONS/EAGLE_PRO_SE_VOICE.md`) and enforcing mandatory pre-service checklists (`LITIGATION/03_DISCOVERY/DISCOVERY_RESPONSE_GATE.md`).

### Four-Layer Documentation Architecture
- **Pattern** (`OPERATIONS/compiled_state_pattern.md`): Theoretical architecture for AI assistant state compilation.
- **Implementation** (`OPERATIONS/EAGLE_IMPLEMENTATION.md`): Concrete divorce-specific pipeline, sources, cron jobs, and CLI specifications.
- **State** (`OPERATIONS/EAGLE_STATE.md`): Daily compiled data read surface (deadlines, comms, tasks, active drafts).
- **Entry** (`AGENTS.md` + `CLAUDE.md`): Operating instructions, safety directives, and authority rules for AI agents entering the repo.

### Authority Hierarchy & Rules
- **Manual Authority Files (Source of Truth)**: `LITIGATION/CASE_STATUS.md` (case facts, deadlines, filings), `SETTLEMENT/FOUNDATIONAL_DATA.md` (locked financial source facts), `SETTLEMENT/SP500_METHODOLOGY.md` (formulas and rationale), `SETTLEMENT/NUMBERS.md` (financial cheat sheet), `NEGOTIATION/DRAFTS/MSA_v7_9_COUNSEL_REVIEW.md` (live operative MSA v7.9).
- **Generated / Non-Authoritative Files**: `OPERATIONS/EAGLE_STATE.md`, `STATUS.md`, `LLM-OVERVIEW.md`. Never cite as source of truth. Manual files always win on conflict.
- **Retired History**: Clio Iris has been completely retired and removed; email, text, and calendar ingestion now route exclusively through the Maya Signal API. Settlement draft versions prior to MSA v7.9 (v5 through v7.8) are superseded. Legacy scripts (`generate_briefing.py`, `eagle_daily_paralegal.py`, `eagle_add_event.py`) have been removed.

## Machine & Host Ownership
The repository operates under a strict **Single-Writer / Multi-Reader** host architecture:

- **Dedicated Writer Machine (`macmini`)**:
  - Hosts the Maya Signal API service (`http://localhost:8200`, port 8200 / Tailscale `100.113.216.27`).
  - Executes the launchd daemon `com.eagle.sync` every 5 minutes (`TOOLS/eagle_bg_sync.sh`) inside `/Users/macmini/.cache/eagle/divorce`.
  - Polls Maya for priority-1 divorce emails, iMessage/SMS, and calendar events, writing normalized updates to `EVIDENCE/comms_emails.md`, `EVIDENCE/comms_texts.md`, and `EVIDENCE/custody_calendar.md`.
  - Executes `poll_lacourt.py --scheduled` (throttled to 6-hour intervals) to poll the Los Angeles Superior Court docket and atomically write `OPERATIONS/LACOURT_DOCKET_STATE.json`.
  - Commits updated normalized records and docket state directly to `origin/main` using the Eagle Bot identity.
- **Interactive Readers & Remote Sandboxes (Claude Code, Codex, Antigravity, phone, browser)**:
  - Read committed runner projections from `origin/main` via `git pull --ff-only`.
  - Execute `./eagle` or `./eagle --daily` locally to compile derived views (`OPERATIONS/EAGLE_STATE.md`, `STATUS.md`).
  - Prohibited from invoking live Maya API endpoints, executing direct live court scrapes, or performing uncoordinated background intake.

## What is actually built

### 1. Eagle Operations Pipeline & State Compiler
- **`eagle` (CLI Orchestrator)**: Shell interface controlling compilation, source staleness checking (`_git_pull_latest`, source file mtime checks against `EAGLE_STATE.md`), JSONL event logging (`./eagle --log`), legal questions, and paralegal outputs.
- **`TOOLS/generate_eagle_state.py`**: Core compiler reading `CASE_STATUS.md`, `TODO.md`, `STANDING_RULES.md`, `comms_emails.md`, `comms_texts.md`, `comms_telegram.md`, `incident_log.md`, `EAGLE_EVENTS.jsonl`, and active draft mtimes to output `OPERATIONS/EAGLE_STATE.md`. Embeds source health tables (`OK`, `STALE`, `MISSING`, `ERROR`) and critical staleness warnings.
- **`TOOLS/gmail_daily_check.py`**: Parses `EVIDENCE/comms_emails.md` to compile `STATUS.md`.
- **`OPERATIONS/EAGLE_EVENTS.jsonl`**: Append-only event ledger tracking non-communication case milestones, filings, and system changes with `observed_at` timestamps and provenance tags (`court`, `email`, `text`, `manual`, `computed`).

### 2. Ingestion Subsystem & Maya Signal Integration
- **Maya Ingestion Drivers (`TOOLS/eagle_imap_sync.py`, `TOOLS/sync_imessage.py`, `TOOLS/sync_calendar.py`)**: Dedicated ingestion scripts called by `eagle_bg_sync.sh` requiring `EAGLE_NATIVE_SYNC=1`. They query Maya endpoints (`/gmail/messages`, `/imessage/messages`, `/calendar/events`) filtering by `scope=divorce&priority=1` (configured via `SHARED/contacts.yaml`). Aborts on sandbox redactions (`[redacted]`).
- **`TOOLS/check_maya_health.py`**: Diagnostic verification script asserting Maya daemon availability, token validity, cache freshness (<3 hours), and validating that retired Clio Iris references have not returned.
- **`TOOLS/poll_lacourt.py`**: Playwright/HTTP poller for LASC docket checks. Maintains atomic portable state in `OPERATIONS/LACOURT_DOCKET_STATE.json`, preserving the last verified snapshot on network failure and logging errors to `TOOLS/.lacourt_last_error`.
- **Discovery Database Ingestion Suite**:
  - `ingest_texts.py`, `ingest_emails.py`, `ingest_claude.py`, `ingest_chats.py`, `ingest_strategy_log.py`, `ingest_transactions.py`, `ingest_chatgpt.py`, `ingest_therapy_notes.py`: Python scripts populating the 187,000+ record SQLite database `divorce_discovery.db`.
  - `schema.sql` & `add_time_views.sql`: Schema definitions and temporal SQL views (`communications_by_month`, `communications_by_week`).
  - `run_full_ingestion.sh` & `run_full_pipeline.sh`: Canonical orchestrators for full database rebuilds.
  - `start_web_platform.sh` & `query.py`: Flask RAG search interface (http://localhost:5000) and command-line SQL summary tool.

### 3. Legal Authority Corpus & Citation Verification Subsystem (`TOOLS/legal_corpus/`)
- **Multi-Tier Authority Model**:
  - **Tier 1**: Hash-anchored official California primary authority (statutes, published case law) with raw source bytes archived in `REFERENCE/LEGAL_CORPUS/` and verified via `./eagle --legal-corpus verify`.
  - **Tier 2**: Labeled secondary authority.
  - **Tier 3**: Unverified research leads sourced via CourtListener API.
- **Corpus Runtime Modules**:
  - `cli.py`: Administrative CLI for corpus verification, gap analysis, and citation sweeps.
  - `verify.py`: Cryptographic SHA-256 validation of archived source bytes, manifests, pinpoints, and issue maps.
  - `tiered_advisor.py` & `advisor.py`: Rule lookup engines answering pointed legal questions via `./eagle --legal-question`. Returns `SUPPORTED` with exact applicability conditions or `UNSUPPORTED` to halt ungrounded legal claims.
  - `draft_auditor.py`: Scans outbound documents (`./eagle --audit-draft <path>`) for Tier 3 or unverified legal citations.
  - `courtlistener_client.py`, `acquire.py`, `graduate_lead.py`: Lead acquisition and promotion pipeline for graduating research leads into Tier 1 authority.
  - SQLite Index: Rebuildable index compiled under `TOOLS/.legal_corpus/`.

### 4. Forensic Financial & Tracing Engine
- **Operative Financial Model (MSA v7.9)**: Master balance sheet integrating S&P 500 benchmarking (`SETTLEMENT/SP500_METHODOLOGY.md`), $869,505.03 retirement pool allocation (with post-separation contribution credits, resolving the float-omar March 2025 contribution discrepancy), real estate valuation, and separate property reimbursement tracking (*Ciprari* floor and S&P ceiling).
- **Float-Omar Forensic Runtime Integration**: Interoperates with preserved forensic runtime at `/Volumes/2TB_SSD/GitHub/.worktrees/float-omar-forensic-design` (branch `codex/forensic-property-reconciliation`, commit `b5d3dd8`, database `trace11.db` with SHA-256 `e6ff2c925a6fe3cc2ba98cd6d7a5be66086743be0b2b265cdd68f8631f3028b1`). Traces 11,236 economic events supporting the $591,493.47 WROS result and $100,047.01 DAF adjustment.
- **Financial Tooling & Analysis Suite (`TOOLS/`)**:
  - `update_sp500.py`: Fetches and updates S&P 500 index historical data for growth multipliers.
  - `analyze_financial_documents.py`, `validate_numbers.py`: Financial balance and integrity checkers.
  - `TOOLS/ingestion/`: Import, OCR, and normalization scripts for Betterment (`betterment_import.py`) and Fidelity (`fidelity_import.py`) financial statements.
  - `TOOLS/analysis/`: Execution pipeline (`fidelity_pipeline_run_all.py`), balance chain trackers, *Ciprari* growth engines (`betterment_ciprari.py`), and 529 optimization models.

### 5. Document Rendering & Settlement Packaging Engine
- **MSA Rendering Engine (`TOOLS/render_msa_document.py`, `generate_msa_review_pdf.py`)**: Markdown-to-PDF compiler generating formal court-ready and settlement documents. Enforces mandatory outbound footer notes (`--footer-note "Settlement proposal under Evid. Code § 1152..."`) to prevent accidental internal draft disclosure.
- **DOCX Settlement Package Generators**:
  - `build_combined_settlement_proposal_docx.py`: Compiles unified settlement proposal packages.
  - `build_wros_proposal_redline_docx.py`: Generates proposal redlines against prior iterations.
  - `build_email_to_counsel_docx.py`: Renders transmittal correspondence for opposing counsel.
  - `build_settlement_term_sheet_docx.py`, `build_wros_checkpoint_schedule_docx.py`, `build_wros_calculation_source_manifest_docx.py`: Builds financial exhibits and source manifests.
- **Settlement Output Structure (`output/send/`)**: Archives outgoing PDF/DOCX transmittal packages (e.g. `2026-09-04_OUTGOING_SETTLEMENT_PACKAGE/`), complete with package indexes, redlines, and signed proposal PDFs.

### 6. Discovery Production & Litigation Response Pipeline
- **Discovery Response Gate (`LITIGATION/03_DISCOVERY/DISCOVERY_RESPONSE_GATE.md`)**: Pre-service checklist enforcing procedural compliance. Mandates signed verifications under penalty of perjury for all substantive discovery responses (*Appleton v. Superior Court*).
- **Set 1 Production Generators (`TOOLS/`)**:
  - `build_set1_service_package.py`, `assemble_set1_signed_responses.py`, `finalize_set1_signed_responses.py`: Assembly scripts for discovery responses.
  - `build_set1_final_bates_production.py` & `pdf_stamp.py`: Sequential Bates-stamping engine.
  - `scan_set1_sensitive_fields.py`, `classify_set1_redaction_candidates.py`, `validate_set1_vision_review.py`, `build_set1_frozen_redacted_derivatives.py`: Redaction management, sensitive field scanning, and vision-model review validation.

### 7. Adversarial Review & Intelligence Tools
- **Adversarial Review Engine (`TOOLS/adversarial_review/`)**: Stress-testing audit tool (`preflight.py`, `checks.py`, `packet.py`, `cli.py`) simulating opposing counsel attacks against proposed filings and financial positions.
- **Intelligence & Analytical Tools**:
  - `temporal_summarizer.py`: Time-windowed (day/week/month) AI event summarization.
  - `narrative_prototype.py`: Chronological narrative builder.
  - `create_comprehensive_timeline.py`: Dual-dating event timeline engine.
  - `dedupe_analyzer.py`: Non-destructive duplicate detection across message corpora.
  - `gottman_analysis.py`: Behavioral and communication pattern analysis.

## Canonical entry points

### Interactive CLI & Operational Commands
- `./eagle`: Standard interactive entry point. Performs staleness check against source files (`CASE_STATUS.md`, `incident_log.md`, `comms_emails.md`, `comms_texts.md`, `LACOURT_DOCKET_STATE.json`, `EAGLE_EVENTS.jsonl`), auto-recompiles if stale, and displays brief.
- `./eagle --daily`: Full daily pipeline (runs local cache checks, updates `STATUS.md`, and compiles `OPERATIONS/EAGLE_STATE.md`).
- `./eagle --refresh`: Recompiles `OPERATIONS/EAGLE_STATE.md` without triggering network sync checks.
- `./eagle --log "<message>"`: Appends a structured event entry to `OPERATIONS/EAGLE_EVENTS.jsonl` and triggers state recompilation.
- `./eagle --legal-question "<question>"`: Evaluates legal assertions against Tier 1 primary authority in the California legal corpus.
- `./eagle --audit-draft <path>`: Audits a draft document for unverified citations and missing authority.
- `./eagle --legal-corpus verify`: Executes cryptographic hash and pinpoint verification across all archived legal corpus files.

### Primary State & Authority Documents
- `OPERATIONS/EAGLE_STATE.md`: Central compiled operational state briefing for AI agents entering the repo.
- `LITIGATION/CASE_STATUS.md`: Manual source of truth for legal posture, case numbers, department, judge, service dates, ATROs, and deadlines.
- `SETTLEMENT/FOUNDATIONAL_DATA.md`: Locked manual source of truth for financial input data and account balances.
- `SETTLEMENT/NUMBERS.md`: Quick-reference working cheat sheet for all financial figures.
- `NEGOTIATION/DRAFTS/MSA_v7_9_COUNSEL_REVIEW.md`: Operative draft of Marital Settlement Agreement v7.9.
- `LITIGATION/03_DISCOVERY/DISCOVERY_RESPONSE_GATE.md`: Pre-service checklist for discovery responses.

### Background Daemons & Ingestion Entry Points
- `TOOLS/eagle_bg_sync.sh`: 5-minute background sync runner executing under `com.eagle.sync` on `macmini`.
- `./run_full_ingestion.sh`: Canonical end-to-end ingest script populating `divorce_discovery.db`.
- `python3 TOOLS/check_maya_health.py`: Health verification script for Maya Signal API connection and authentication.
- `python3 TOOLS/legal_corpus/cli.py`: Standalone CLI for managing, expanding, and auditing the California legal corpus.
