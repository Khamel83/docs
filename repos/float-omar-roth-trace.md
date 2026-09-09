# LLM-OVERVIEW — float-omar-roth-trace
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is

`float-omar-roth-trace` is the authoritative personal finance repository, forensic account tracing system, Marital Settlement Agreement (MSA v7.1) reconciliation engine data store, and family capital planning hub for Omar. It was split out of `khamel83/float` on 2026-07-24 (following `float`'s ADR 0002) to isolate personal financial data, account inventories, statement indices, forensic divorce tracing reports, and capital baselines from the core application codebase.

The repository maintains canonical financial accounting ledgers (`canonical/`), account master configurations and planning documents (`omar/`), forensic divorce reconciliation specifications (`docs/`), database schema migration history (`migrations_archive/`), audit and ledger gate scripts (`scripts/`), and historical source archives from the `float` monorepo (`archive/float-source/`).

## Machine & Host Ownership

No multi-host or explicit host ownership information is available in the repository status probe, code configurations, or `AGENTS.md`.

## What is actually built

### 1. Omar Personal Capital & Account Master System (`omar/`)
- **Family Capital Baseline**: `omar/planning/FAMILY_CAPITAL_MASTER.md` serves as the authoritative source for capital planning, assumptions, trust structures, and retirement models, managed alongside `omar/planning/PLANNING_README.md`.
- **Account Identity & Inventory Master**: `omar/ACCOUNT_MASTER.md`, `omar/account_identity_master.csv`, `omar/account_inventory_raw.csv`, `omar/account_master.csv`, and `omar/plaid_items_raw.json` establish account mappings, identity attributes, and Plaid item bindings.
- **Statement & Ingest Management**: `omar/statement_index/`, `omar/input/`, `omar/backups/`, and `omar/thoughts/` manage raw financial statement manifests, historical backups, and working analysis notes.

### 2. Canonical Ledger & Forensic Tracing Engine (`canonical/` & `docs/`)
- **Canonical Accounting Data**: `canonical/canonical_events.csv`, `canonical/accounting_legs.csv`, `canonical/accepted_ledger`, `canonical/accepted_ledger_acceptance_summary.csv`, and `canonical/acceptance_residuals.csv` store normalized financial transaction events, accounting legs, and gate acceptance records.
- **Lineage & Transit Mapping**: `canonical/account_lineage`, `canonical/account_type_summary.csv`, `canonical/ambiguous_pairs.csv`, `canonical/balance_observations.csv`, and `canonical/clearing_transit_summary.csv` track account transformations, clearing transits, and balance audits.
- **Divorce Tracing & MSA v7.1 Documentation**: `docs/DIVORCE_TRACING_REPORT_SPEC.md`, `docs/DIVORCE_TRACING_WIREFRAME.html`, `docs/CHRONOLOGICAL_RECONCILIATION_AUDIT.md`, `docs/FINANCIAL_CORPUS_INDEX.md`, and `docs/RECONCILIATION_GAPS.md` define legal/forensic specifications, reciprocal retirement division models, binary house authority, and adversarial reconciliation packages. Forensic audit suites reside in `docs/forensics/`, `docs/reconciliation/`, `docs/instructions/`, `docs/archive/`, and `docs/screenshots/`.

### 3. Reconciliation, Audit & Analysis Scripts (`scripts/` & `analysis_scripts/`)
- **Ledger Acceptance & Reconciliation**: `scripts/build-acceptance-gate.ts` (with `build-acceptance-gate.test.ts`) and `scripts/build-accepted-ledger.test.ts` validate ledger entries against strict acceptance gates. `scripts/apply-non-asset-reconciliation.ts` executes non-asset reconciliation adjustments.
- **Forensic Auditing & Basis Tracking**: `scripts/audit-statement-trace-variance.ts` computes variance across statement traces; `scripts/audit-betterment-holding-change-scope.ts` audits holding scope changes; `scripts/backfill-cost-basis.ts` (with `backfill-cost-basis.test.ts`) backfills asset cost basis.
- **Performance Benchmarking**: `scripts/beat-the-market.ts` (with `beat-the-market.test.ts`) calculates portfolio returns against market indexes.
- **Legacy Analysis**: `analysis_scripts/divorce_legacy` stores legacy divorce tracing scripts.

### 4. Database Schema & Migration Archive (`migrations_archive/`)
- Archives SQLite database migrations tracing schema evolution: baseline schema (`000_baseline.sql`, `001_initial_schema.sql`), liabilities and real estate (`007_liabilities_real_estate.sql`), stock pricing (`011_stock_prices.sql`), mortgage terms (`014_mortgage_terms.sql`), category remapping (`017_remap_legacy_categories.sql`, `018_consolidate_copilot_categories.sql`, `024_remap_mint_categories.sql`), Plaid metadata (`019_plaid_transaction_account_metadata.sql`), and childcare category restoration (`023_restore_childcare_category.sql`).

### 5. Archived Float Source Modules (`archive/float-source/`)
- **Gmail Ingress MCP Server**: `archive/float-source/python/gmail_mcp/server.py` implements a FastMCP server (`float-gmail-ingress`) using OAuth (`auth.py`) to scrape financial notifications from senders including Chase, Amex, PayPal, Venmo, Capital One, Discover, and Mastercard.
- **Retirement Calculator Frontend**: `archive/float-source/frontend/retirement-calculator/retire/engine.js` provides DOM-based financial projections covering taxable/non-taxable portfolios, real estate equity, annual income, savings, and monthly spend.
- **Source Manifest Builder**: `archive/float-source/scripts/build-source-manifest.ts` generates `canonical/masters/source_document_master.csv` to ensure declared source documents match `raw_observations` counts.
- **Redfin AVM Ingest**: `archive/float-source/scripts/fetch-redfin-estimate.ts` queries Redfin's AVM API for property ID `7090486`, handling Redfin's `{}&&` JSON anti-hijacking prefix and storing estimates via `src/db/client.ts`.

### 6. Session & Project Context (`agent_docs/`, root files)
- `agent_docs/latest_session_work.md` and `agent_docs/project_progress.md` record agent work history and task state.
- `CLAUDE.md`, `CONTEXT.md`, `CONTRACT.md`, `HANDOFF.md`, and `tmp_test_1099.csv` define repo handoff procedures and test data contracts.

## Canonical entry points

- **Family Capital Baseline**: `omar/planning/FAMILY_CAPITAL_MASTER.md` (read `omar/planning/PLANNING_README.md` first).
- **Account Identity & Inventory Master**: `omar/ACCOUNT_MASTER.md` & `omar/account_master.csv`.
- **Reconciliation & Acceptance Gate Pipeline**: `scripts/build-acceptance-gate.ts`, `scripts/apply-non-asset-reconciliation.ts`.
- **Audit & Cost Basis Backfill**: `scripts/audit-statement-trace-variance.ts`, `scripts/backfill-cost-basis.ts`.
- **Divorce Tracing Spec & Forensic Audit**: `docs/DIVORCE_TRACING_REPORT_SPEC.md`, `docs/CHRONOLOGICAL_RECONCILIATION_AUDIT.md`.
- **Canonical Accounting Ledger Output**: `canonical/accepted_ledger`, `canonical/canonical_events.csv`, `canonical/accounting_legs.csv`.
- **Database Schema Archive**: `migrations_archive/001_initial_schema.sql`.
- **Gmail Ingress MCP Server**: `archive/float-source/python/gmail_mcp/server.py`.
