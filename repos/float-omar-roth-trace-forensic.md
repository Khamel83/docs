# LLM-OVERVIEW — float-omar-roth-trace-forensic
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
`float-omar-roth-trace-forensic` (package name `float-omar-forensics`) is a private forensic financial workspace and marital property reconciliation engine for Omar, split out from `khamel83/float`. It computes separate property (SP) vs. community property (CP) characterization, asset tracing, and reimbursement accounting under California legal rules for the marriage period starting `2016-05-28` through the legal Date of Separation (DOS) `2025-03-21` (with `2025-03-31` serving as the statement/retirement endpoint).

Core forensic design constraints:
- **Governing Specs**: `docs/superpowers/specs/2026-08-02-forensic-separate-community-property-reconciliation-design.md` and `docs/superpowers/plans/2026-08-02-forensic-property-reconciliation-implementation.md`.
- **Immutability & Safety**: Original evidence bytes and vault sources are read-only. Generated databases and deliverables stay under ignored run directories. Account numbers and private financial contents are excluded from routine messages.
- **Key Financial Facts**: The $183,012.33 house withdrawal leaves the brokerage pool exactly once; the $212,000 escrow wire is a distinct financial fact.
- **Legal Characterization Rules**: Retitling separate property jointly or depositing into a joint account does not constitute an automatic gift or transmutation. Characterization follows Family Code § 852 (writing requirement), Probate Code § 5305 (tracing qualifying multiple-party deposit accounts), and Family Code §§ 2581/2640 (reimbursement predicates). Retirement assets are tracked as a separately disclosed benchmark silo.
- **Execution & Security Gate**: Maximum unattended state is `COMPUTATION_READY`. Transition to `FINAL_FOR_EXTERNAL_USE` requires explicit authorization from Omar. Dual-route execution uses Sol for plan/Git state orchestration, Luna executors for bounded production packages, and separate verifier roles (primary and verifier extractions are strictly segregated across workers).

## Machine & Host Ownership
No multi-host or host-specific network configuration is specified in `AGENTS.md`, status probe output, or repository configurations; execution runs on a local Node 22 workspace (`>=22 <23`).

## What is actually built
The repository consists of a Node.js/TypeScript forensic accounting engine, SQLite database builders, canonical CSV ledgers, forensic configurations, and archived legacy components:

- **TypeScript Forensic Accounting Engine (`src/`, `scripts/forensics/`, `scripts/`)**:
  - `src/forensics` & `src/pipelines`: Core tracing algorithms, statement tracing pipelines, and ledger reconciliation modules.
  - `scripts/forensics/run-primary.ts` & `run-verifier.ts`: Primary and Verifier execution scripts enforcing clean-room segregation between primary data extraction and verifier verification.
  - `scripts/forensics/build-evidence-database.ts`: Compiles raw financial observations into a SQLite database using `better-sqlite3` (v12.10.0).
  - `scripts/forensics/build-source-manifest.ts`: Generates `canonical/masters/source_document_master.csv` to validate ingested raw observations against declared sources.
  - `scripts/forensics/build-event-ledger.ts`: Generates canonical event ledgers.
  - `scripts/forensics/build-reconciliation-report.ts`, `build-final-reconciliation.ts`, `refresh-reconciliation.ts`: Builds and deterministically refreshes forensic reconciliation reports and packages.
  - `scripts/forensics/promote-runtime.ts` & `build-release.ts`: Handles runtime artifact promotion and release packaging.
  - `scripts/forensics/build-x63-source-register.ts`: Registers X63 source document inputs.
  - Operational scripts: `apply-non-asset-reconciliation.ts`, `audit-betterment-holding-change-scope.ts`, `audit-statement-trace-variance.ts`, `backfill-cost-basis.ts`, `beat-the-market.ts`, `build-acceptance-gate.ts`, and `build-accepted-ledger.ts`.

- **Canonical Data Layer (`canonical/`, `omar/`)**:
  - `canonical/`: Versioned reconciliation ledgers and analytical summaries (`accepted_ledger`, `canonical_events.csv`, `accounting_legs.csv`, `account_type_summary.csv`, `acceptance_residuals.csv`, `accepted_ledger_acceptance_summary.csv`, `ambiguous_pairs.csv`, `balance_observations.csv`, `clearing_transit_summary.csv`, `account_lineage`).
  - `omar/`: Authoritative master planning baseline (`omar/planning/FAMILY_CAPITAL_MASTER.md`), raw account indices and inventories (`account_master.csv`, `account_identity_master.csv`, `account_inventory_raw.csv`), raw Plaid metadata (`plaid_items_raw.json`), statement indices (`statement_index/`), input statements, backups, and scratch notes (`thoughts/`).

- **Forensic Configuration (`config/`)**:
  - `config/forensic-property-reconciliation.json`: Primary configuration governing property reconciliation parameters.
  - `config/roth-ira-characterization.json`: Specific tracing and characterization rule definitions for Roth IRA accounts.
  - `config/current-valuation-sources.json` & `retirement-silo-sources.json`: Asset valuation mappings and retirement benchmark silo source mapping.

- **Database Schemas (`migrations_archive/`)**:
  - SQL schema migrations (`000_baseline.sql` through `024_remap_mint_categories.sql`) covering transaction categorization, liabilities/real estate (`007`), stock prices (`011`), mortgage terms (`014`), and Plaid metadata (`019`).

- **Deliverables & Specifications (`deliverables/`, `artifacts/`, `docs/`, `templates/`)**:
  - Forensic deliverables, output templates (`templates/forensic-property-reconciliation`), and technical specifications (`docs/DIVORCE_TRACING_REPORT_SPEC.md`, `CHRONOLOGICAL_RECONCILIATION_AUDIT.md`, `FINANCIAL_CORPUS_INDEX.md`, `RECONCILIATION_GAPS.md`).

- **Archived / Retired Legacy Subsystems (`archive/`, `analysis_scripts/`)**:
  - `archive/float-source/frontend/retirement-calculator`: Retired browser-based JavaScript retirement calculator interface.
  - `archive/float-source/python/gmail_mcp`: Retired Python Gmail MCP ingress server (`FastMCP`, Google OAuth) for financial email ingestion.
  - `analysis_scripts/divorce_legacy`: Legacy divorce analysis scripts superseded by the TypeScript pipeline.

## Canonical entry points
All pipeline commands run via `npm` scripts requiring Node 22 (`>=22 <23`):

- **Environment & Type Verification**:
  - `npm run preflight:node`: Verifies host is running Node 22 (`node -e "..."`).
  - `npm run typecheck`: Executes Node preflight check and TypeScript verification (`tsc --noEmit`).

- **Test Execution**:
  - `npm test`: Runs Vitest test suite in forks pool (`vitest run --pool=forks` covering `src/**/*.test.ts` and `scripts/forensics/**/*.test.ts`).
  - `npm run test:forensic`: Target test run for forensic pipelines in `src/pipelines/statement-trace`, `src/forensics`, and `scripts/forensics`.

- **Forensic Pipeline Execution Scripts**:
  - `npm run forensic:manifest`: `tsx scripts/forensics/build-source-manifest.ts`
  - `npm run forensic:evidence-db`: `tsx scripts/forensics/build-evidence-database.ts`
  - `npm run forensic:x63-source-register`: `tsx scripts/forensics/build-x63-source-register.ts`
  - `npm run forensic:events`: `tsx scripts/forensics/build-event-ledger.ts`
  - `npm run forensic:primary`: `tsx scripts/forensics/run-primary.ts`
  - `npm run forensic:verifier`: `tsx scripts/forensics/run-verifier.ts`
  - `npm run forensic:reconciliation-report`: `tsx scripts/forensics/build-reconciliation-report.ts`
  - `npm run forensic:final-reconciliation`: `tsx scripts/forensics/build-final-reconciliation.ts`
  - `npm run forensic:refresh-reconciliation`: `tsx scripts/forensics/refresh-reconciliation.ts`
  - `npm run forensic:promote`: `tsx scripts/forensics/promote-runtime.ts`
  - `npm run forensic:release`: `tsx scripts/forensics/build-release.ts`
