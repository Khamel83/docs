# LLM-OVERVIEW — divorce-gmail-reliability
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
This repository hosts the divorce discovery, communication intelligence, legal timeline integration, and settlement generation platform for *Omar Zoheri v. Meghan Logue*. It ingests multi-channel communications (over 187,000 to 195,000+ records spanning 2014–2025 across iMessage, SMS, Email, Google Chat, and Claude AI strategy logs) into a central SQLite database (`divorce_discovery.db`). The system runs an automated state-compilation pipeline ("Eagle") to produce daily operational briefings, PDF exhibit packages, and litigation risk/timeline analysis.

Data collection (Phase 2) is complete, with active operations focused on Phase 3 timeline integration, evidence verification, and background synchronization (Gmail IMAP, Clio, and custody calendar sync). 

Safety and authority hierarchy rules strictly govern this codebase:
- `LITIGATION/CASE_STATUS.md` is the manual source of truth for legal facts, deadlines, and financial figures, overriding all generated state files (`OPERATIONS/EAGLE_STATE.md`, `STATUS.md`).
- Files under `EVIDENCE/`, `SETTLEMENT/`, `LITIGATION/`, and `NEGOTIATION/` are protected read-only litigation records.

## Machine & Host Ownership
- **Homelab Service Unit**: Configured in `homelab.yaml` under project ID `divorce`, managed as a `service` lifecycle in `development` mode with telemetry routed to `homelab-api:/observations`.
- **Remote Runner Host**: Defined in `Makefile` as `EAGLE_RUNNER ?= homelab` pointing to remote repository path `/home/khamel83/github/divorce`. Python ingestors and tools (`TOOLS/import_gchat_json.py`) reference paths anchored at `/home/khamel83/github/divorce/`.
- **Stateless VM Synchronization**: `eagle_daily_commit.sh` commits state and `.eagle_cache.json` to `origin/main` to support headless/stateless VM execution.
- **Status Probe**: Status probe output disabled (requires `JANITOR_RUN_STATUS_PROBE=1`).

## What is actually built

### 1. Database & Core Data Store
- **`divorce_discovery.db`**: Primary SQLite database holding structured tables:
  - `communications`: 187,058+ (up to 195,796+) records (113,662 iMessages, 71,757 SMS, 1,592 Emails, Google Chat JSON records, and 47 Claude AI sessions).
  - `legal_strategy_events`: 145+ catalogued strategy events.
  - `file_manifest`: Manifest tracking 26+ ingested source files.
  - Financial separation date bound: May 8, 2025.

### 2. Ingestion & Communication Pipeline
- **Root Python Ingestors**: `ingest_texts.py`, `ingest_emails.py`, `ingest_claude.py`, `ingest_chats.py`, `ingest_strategy_log.py` populate `divorce_discovery.db`.
- **Google Chat Importer**: `TOOLS/import_gchat_json.py` parses Takeout exports (`messages.json`), mapping senders using identity configurations (`OMAR_EMAILS` vs `MEGHAN_EMAILS`).
- **Sync & Reliability Suite**:
  - `test_eagle_bg_sync.py` & `test_eagle_imap_sync.py`: Background Gmail and IMAP synchronization.
  - `GMAIL_SYNC_RELIABILITY_SPEC.md`: Operational spec establishing fail-closed runtime token precedence and zero-data-loss Gmail sync rules.
  - Clio Sync (`codex/eagle-clio-sync-hardening`): Clio legal management integration and daily state commit routines.

### 3. Intelligence, Analysis & Timeline Engine
- **`temporal_summarizer.py`**: AI-driven daily, weekly, and monthly communication summarizer.
- **`narrative_prototype.py`**: Story-first litigation intelligence generator.
- **`create_comprehensive_timeline.py`**: Dual-dating timeline synthesis linking communication logs with case milestones.
- **`dedupe_analyzer.py`**: Non-destructive duplicate message detector.
- **`TOOLS/analyze_financial_documents.py`**: Extracts PDF metadata and asset protection evidence across `financial_pre_marriage` and `tax_documents`.
- **`TOOLS/analyze_therapy_conversations.py`**: Semantic analysis of Claude AI therapy and strategy logs.
- **`test_corpus_filter.py`**: Script validating cursor iteration and record filtering over SQLite datasets.

### 4. Eagle State & Briefing Compilation Pipeline
- **`./eagle`**: Core CLI wrapper for briefing generation and pipeline execution.
- **`TOOLS/generate_eagle_state.py`**: Compiles `OPERATIONS/EAGLE_STATE.md` and public summary `STATUS.md`.
- **`TOOLS/eagle_daily_paralegal.py`**: Generates `OPERATIONS/EAGLE_PARALEGAL.md`.
- **`TOOLS/validate_structure.py`**: Structural validator checking file feed freshness and pipeline integrity.
- **`OPERATIONS/EAGLE_EVENTS.jsonl`**: Ledger recording sync and operational events.

### 5. Exhibit Generation & System Documentation
- **`TOOLS/exhibit_generation_engine.py`**: Professional PDF exhibit compiler using ReportLab and Matplotlib, outputting structured packages to `exhibit_packages/`.
- **`TOOLS/complete_system_documentation.py`**: Generates comprehensive system rerun specs and hash verification reports (`COMPLETE_SYSTEM_DOCUMENTATION`).

### 6. Marcus Web & OAuth Subsystem (`MARCUS/`)
- Web application (`app`), database migrations (`migrations`), legal doctrine (`doctrine`), documentation (`docs`), and OAuth authorization handlers (`gen_oauth_url.py`, `oauth_exchange.py`).

### 7. Litigation & Case Storage Organization
- **`EVIDENCE/`**: Source extracts (Betterment, Fidelity, Zakat 2026, daycare/email/telegram logs).
- **`LITIGATION/`**: Filings (`00_FILING_PACKAGE`), disclosures (`01_DISCLOSURES`), motions (`02_MOTIONS`), discovery (`03_DISCOVERY`), custody (`04_CUSTODY`), timelines (`05_TIMELINE`), judicial research (`JUDGE_BYRDSONG_RESEARCH.md`), and manual source of truth `CASE_STATUS.md`.
- **`SETTLEMENT/`**: Foundational balance sheets (`NUMBERS.md`, `FOUNDATIONAL_DATA.md`), dissipation claims (`DISSIPATION_CLAIM.md`), contingency options (`OPTION_5_CONTINGENCY.md`), and formal settlement proposals.
- **`NEGOTIATION/`**: Settlement models (`SETTLEMENT_PLAN_2026-07.md`), education funding walkthroughs, negotiation filters, and simulation outputs.
- **`PRIVATE/`**: Confidential strategy files (`katie-loren.md`, `three_incidents.md`), guidance, and notes.
- **`REFERENCE/`**: California Family Code reference, Judicial Council forms (SB1427), and pro se guidance.
- **`SHARED/` & `SOURCE_DOCS/`**: Common assets (`contacts.yaml`, `standing_rules.yaml`), financial paystubs, bank transfers, and raw statement PDFs.

## Canonical entry points

| Purpose | Entry Point / Command | Description |
|---|---|---|
| **Primary CLI** | `./eagle` or `./eagle --brief` | Shell orchestrator to run pipeline or print compiled state brief. |
| **Pipeline Runner** | `make eagle-run` | Executes paralegal compiler and state generator targets via `PYTHON=python3.12`. |
| **State Generation** | `make eagle-state` | Runs `TOOLS/generate_eagle_state.py` to update `OPERATIONS/EAGLE_STATE.md`. |
| **Paralegal Briefing** | `make eagle-paralegal` | Runs `TOOLS/eagle_daily_paralegal.py` to update `OPERATIONS/EAGLE_PARALEGAL.md`. |
| **Structure Validation** | `make eagle-validate` | Runs `TOOLS/validate_structure.py` to verify feed freshness and required files. |
| **Event Ledger** | `make eagle-events` | Tail inspection of `OPERATIONS/EAGLE_EVENTS.jsonl`. |
| **Remote Runner Check** | `make eagle-runner-check` | SSH check against `homelab` remote git and cron configuration. |
| **Remote State Run** | `make eagle-runner-state` / `make eagle-runner-run` | SSH execution of state compilation and daily pipeline on `homelab`. |
| **Full Ingestion** | `./run_full_ingestion.sh` | Shell wrapper chaining Python message and communication ingestors. |
| **Master Tool Runner** | `TOOLS/RUN_EVERYTHING.sh` | Master execution shell script for intelligence tool suites. |
| **Web App Launch** | `./start_web_platform.sh` | Launches web interface platform. |
| **State Sync Script** | `./eagle_daily_commit.sh` | Commits generated state and cache to `origin/main` for remote VM workers. |
| **Manual Source of Truth** | `LITIGATION/CASE_STATUS.md` | Primary manual authority for legal facts, deadlines, and figures. |
| **Documentation Entry** | `AGENTS.md` & `CLAUDE.md` | Repository constitution, authority rules, and AI interaction guidelines. |
