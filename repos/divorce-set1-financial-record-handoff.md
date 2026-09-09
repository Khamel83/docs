# LLM-OVERVIEW — divorce-set1-financial-record-handoff
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
`divorce-set1-financial-record-handoff` (repo identifier `divorce`) is an operational legal discovery, evidence processing, financial analysis, and state compilation system ("Eagle System") for *Omar Zoheri v. Meghan Logue*. The repository ingests multi-channel communications (iMessage, SMS, email, Claude AI therapy/strategy sessions, Telegram), financial records (Betterment, Fidelity, Ally Bank, paystubs, IRAs), legal strategy events, and California Judicial Council disclosure forms into a unified SQLite database (`divorce_discovery.db`). It automatically compiles authoritative operational briefings (`OPERATIONS/EAGLE_STATE.md`, `STATUS.md`), generates Bates-stamped PDF exhibit packages, validates discovery packets against California procedural rules, and hosts the MARCUS AI paralegal subsystem. Manual litigation facts and legal deadlines are anchored in `LITIGATION/CASE_STATUS.md`, while immutable litigation records reside under `EVIDENCE/`, `SETTLEMENT/`, `LITIGATION/`, and `NEGOTIATION/`.

## Machine & Host Ownership
- **Primary Execution Host**: Remote runner host `homelab` (path `/home/khamel83/github/divorce`), configured via `Makefile` (`EAGLE_RUNNER ?= homelab`, `REMOTE_REPO ?= /home/khamel83/github/divorce`) for remote state generation, daily cron pipelines, and runner health checks.
- **Service Configuration**: Configured in `homelab.yaml` as project ID `divorce` (repository `https://github.com/Khamel83/divorce`, kind `service`, owner `homelab`), with runtime unit `divorce` and observability sink `homelab-api:/observations`.
- **Background Synchronization**: Automated crons and background jobs (`[eagle_bg_sync]`) execute Gmail synchronization (`OPERATIONS/GMAIL_SYNC_RELIABILITY_SPEC.md`), Clio sync (`eagle-clio-sync-hardening`), docket synchronization, and daily git state commits (`eagle_daily_commit.sh`) to maintain state persistence across environments.

## What is actually built
- **Core Database & Schema (`divorce_discovery.db`)**:
  - `communications`: Holds 195,796+ indexed records (iMessage: 113,662; Text/SMS: 71,757; Email: 1,592; Claude AI sessions: 47) spanning 2014–2025.
  - `legal_strategy_events`: 145+ catalogued critical case timeline events and legal strategy milestones.
  - `file_manifest`: 26+ tracked source files and ingestion manifests.
  - Views and SQL utilities: `schema.sql` table structures and `TOOLS/add_time_views.sql` temporal query layers.
- **Eagle Operational Engine & State Compilation**:
  - CLI wrapper (`./eagle`): Orchestrates state generation and legal authority verification (`./eagle --legal-question "<question>"`), enforcing fail-closed compliance against governing legal sources before asserting rules.
  - State Generators (`TOOLS/generate_eagle_state.py`, `TOOLS/eagle_daily`): Compiles operational state into `OPERATIONS/EAGLE_STATE.md`, `OPERATIONS/EAGLE_PARALEGAL.md`, and `STATUS.md`.
  - Sync & Reliability Pipeline: Specs and scripts for Gmail sync (`OPERATIONS/GMAIL_SYNC_RELIABILITY_SPEC.md`), Docket sync (`OPERATIONS/DOCKET_SYNC_RELIABILITY_SPEC.md`), Clio sync hardening, LACourt polling (`tests/test_poll_lacourt.py`), IMAP sync (`tests/test_eagle_imap_sync.py`), and automated daily git state commits (`eagle_daily_commit.sh`).
- **Ingestion & Extraction Subsystems**:
  - Communication Ingestors: Python modules `ingest_texts.py`, `ingest_emails.py`, `ingest_claude.py`, `ingest_chats.py` executed individually or chained via `run_full_ingestion.sh`.
  - Conversation Extraction Pipeline: Multi-source extraction pipeline configured in `corpus_extraction_pipeline/` and validated by `TOOLS/validate_extraction_setup.py`.
  - Financial Data Extraction: Parsed financial records in `EVIDENCE/` (Betterment extract batches 1–3, Fidelity statements, Zakat 2026 calculations) and raw source documents in `SOURCE_DOCS/` (Ally Bank wire transfers, Meghan Rollover IRA spreadsheets, paystubs, account balance snapshots).
- **Validation, Legal Analysis & Exhibit Generation (`TOOLS/`)**:
  - `TOOLS/exhibit_generation_engine.py`: PDF exhibit generator utilizing ReportLab, matplotlib, and pandas to compile court-ready exhibit packages in `exhibit_packages/`.
  - `TOOLS/pdf_stamp.py`: Image-based Bates stamping utility (`clear_bates_zone`, `draw_bates_stamp`) operating in 200 DPI pixel-space.
  - `TOOLS/validate_discovery_packet.py`: Defect scanner for discovery packet drafting (verifies FL-145 interrogatories, cumulative special interrogatory limits).
  - System Verification & Analysis Tools: `TOOLS/complete_system_documentation.py`, `TOOLS/FINAL_ANALYSIS_TOOLKIT.py`, `TOOLS/activity_log_v2.py`, `TOOLS/analyze_financial_documents.py`, `TOOLS/analyze_divorce_document_handoff.mjs`, `TOOLS/RUN_EVERYTHING.sh`.
- **MARCUS AI Subsystem (`MARCUS/`)**:
  - Modular AI agent environment containing application service (`MARCUS/app`), database migration engine (`MARCUS/migrations`), OAuth flow scripts (`gen_oauth_url.py`, `oauth_exchange.py`), parenting plan doctrine rules (`PARENTING_PLAN_RULES.md`), and agent narrative guidelines (`AGENT_NARRATIVE.md`).
- **Litigation & Settlement Directory Structure**:
  - `LITIGATION/`: Structured legal records including filing packages (`00_FILING_PACKAGE`), disclosures (`01_DISCLOSURES`), motions (`02_MOTIONS`), discovery requests (`03_DISCOVERY`), custody logs (`04_CUSTODY`), timelines (`05_TIMELINE`), argument vectors, case status (`CASE_STATUS.md`), and judicial research (`JUDGE_BYRDSONG_RESEARCH.md`).
  - `SETTLEMENT/`: Dissipation claim calculations (`DISSIPATION_CLAIM.md`), foundational financial disclosures, gap ledgers (`RETIREMENT_FACT_AND_GAP_LEDGER.md`), and contingency options.
  - `NEGOTIATION/`: Settlement plans (`SETTLEMENT_PLAN_2026-07.md`), education funding walkthroughs, domain configurations, and negotiation filters.
  - `REFERENCE/`: Statutory rules (`CALIFORNIA_FAMILY_CODE.md`), legal corpus files, pro se guidance, and Judicial Council form indexes (`SB1427_judicial_council_forms.md`).
  - `PRIVATE/`: Strategy notes, guidance documents, and key incident logs.
  - `SHARED/`: Common identity files, contact definitions (`contacts.yaml`), and standing procedural rules (`standing_rules.yaml`).

## Canonical entry points
- `./eagle`: Core operational CLI wrapper for state compilation and legal question validation (`./eagle --legal-question "<query>"`).
- `Makefile`:
  - `make eagle-run`: Processes recent events and updates state.
  - `make eagle-state`: Executes `TOOLS/generate_eagle_state.py` to regenerate `OPERATIONS/EAGLE_STATE.md`.
  - `make eagle-paralegal`: Executes `TOOLS/eagle_daily` to update `OPERATIONS/EAGLE_PARALEGAL.md`.
  - `make eagle-validate`: Verifies feed freshness and required state files.
  - `make eagle-legal-build` / `make eagle-legal-verify` / `make eagle-legal-status`: Builds and verifies the legal corpus search index.
  - `make eagle-runner-run` / `make eagle-runner-state` / `make eagle-runner-check`: Dispatches execution commands to the remote `homelab` runner host via SSH.
- `run_full_ingestion.sh`: Main ingestion orchestrator executing Python ingestors (`ingest_texts.py`, `ingest_emails.py`, `ingest_claude.py`, `ingest_chats.py`).
- `start_web_platform.sh`: Shell launcher for the web discovery interface.
- `TOOLS/RUN_EVERYTHING.sh`: Master pipeline execution script for data extraction and analysis.
- `TOOLS/validate_discovery_packet.py`: Discovery packet drafting validator (`python3 TOOLS/validate_discovery_packet.py <packet.md> [--previous-special-count N]`).
- `MARCUS/gen_oauth_url.py` & `MARCUS/oauth_exchange.py`: Auth handlers for MARCUS agent OAuth integration.
- `test_corpus_filter.py` & `tests/`: Test suite runner entrypoints covering legal corpus filters, Maya signal sync, Eagle sync/deadlines/docket/imap, and discovery packet fact sources.
