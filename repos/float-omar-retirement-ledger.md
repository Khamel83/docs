# LLM-OVERVIEW — float-omar-retirement-ledger
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
`float-omar-retirement-ledger` is Omar's private financial repository containing account master ledgers, post-divorce retirement feasibility modeling, financial statement indexing, and canonical reconciliation datasets. Split from `khamel83/float` on 2026-07-24, it isolates personal wealth tracking, trust planning (`omar/planning/FAMILY_CAPITAL_MASTER.md`), and divorce financial tracing from the core `float` software engine.

The repository serves as the single source of truth for post-divorce MSA v7.1 asset division feasibility modeling, tax-aware retirement cash-flow engine calculations, historical account master inventory, and canonical acceptance ledgers.

## Machine & Host Ownership
No multi-host information is available in the repository configurations or status probe output.

## What is actually built

### 1. Tax-Aware Retirement Feasibility Engine (`omar/planning/retirement_feasibility/`)
A Python engine modeling post-divorce retirement spending, investment cash flows, tax obligations, and Monte Carlo portfolio survival.
- **`model.py`**: Annual account cash-flow engine modeling taxable, taxable basis, traditional IRA/401(k), Roth IRA, HSA, and cash balances (`Accounts` dataclass). Implements RMD schedules (`RMD_DIVISORS` table for ages 75–100), tax-aware portfolio withdrawal ordering, ACA MAGI guardrails, pre/post-65 healthcare expenses, and root-finding tax sales solvers (`_required_taxable_sale`).
- **`tax.py`**: Income tax solver (`TaxRules`, `tax_bill`) supporting ordinary income brackets, long-term capital gains tiers, Net Investment Income Tax (NIIT 3.8%), flat state income tax, indexed standard deductions, and Social Security taxability rules.
- **`scenario.py`**: Data structure (`Scenario`) parsing raw JSON scenarios (`2026-08-22-retirement-scenario.json`). Builds initial account balances across trust allocation cases (e.g. `base`, `no_family_trust`, `no_family_trust_pretax_cutover`).
- **`analysis.py`**: Deterministic projections (`deterministic_projection`) and lognormal geometric Monte Carlo simulations (`monte_carlo_result`), solving for lifestyle spending capacity and success rates across age horizons.
- **`reporting.py`**: Structured summary and Markdown output generator for scenario runs.
- **`runner.py`**: Reproducible scenario executor (`analyze_case`) conducting multi-dimensional parameter sweeps across retirement ages (52–65), return rates, cost basis ratios, and Social Security availability.
- **`render_report.py`**: Command-line entry point generating comprehensive retirement plan artifacts in HTML, Markdown, and JSON formats (`2026-08-22-comprehensive-retirement-plan.html`, `.md`, `.json`, `2026-retirement-feasibility-post-divorce-scenario.html`).
- **`tests/`**: Pytest test suite validating model mechanics, tax rules, scenario parsing, analysis functions, and reporting (`test_model.py`, `test_tax.py`, `test_scenario.py`, `test_analysis.py`, `test_reporting.py`, `test_runner.py`).

### 2. Account Master & Capital Planning (`omar/`)
- **Master Files**: `ACCOUNT_MASTER.md`, `account_master.csv`, `account_identity_master.csv`, and `account_inventory_raw.csv` defining account lineage, institution metadata, and balance tracking.
- **Planning**: `omar/planning/` containing `FAMILY_CAPITAL_MASTER.md` (authoritative family capital allocation), trust structures, and legacy planning tools.
- **Statement Indexing (`omar/statement_index/`)**: Python indexing tools (`discover.py`, `extract.py`, `classify.py`, `documents_needed.py`, `resolve_accounts.py`, `load_known.py`) that catalog incoming financial statements from `omar/input/`.
- **Raw Data & Backups**: `omar/input/`, `omar/backups/`, `omar/thoughts/`, and `plaid_items_raw.json` storing raw institution records and Plaid connection metadata.

### 3. Canonical Accounting Ledger Corpus (`canonical/`)
Snapshot CSV ledgers exported from the reconciliation pipeline:
- `canonical_events.csv`, `balance_observations.csv`, `accounting_legs.csv`, `accepted_ledger_acceptance_summary.csv`, `account_lineage`, `account_type_summary.csv`, `clearing_transit_summary.csv`, `acceptance_residuals.csv`, and `ambiguous_pairs.csv`.

### 4. Reconciliation, Audit & Pipeline Tooling (`scripts/`)
TypeScript (`tsx`) and Python scripts executing ledger validation and financial tracing:
- **Ledger Generation**: `publish-accepted-ledger.ts`, `build-accepted-ledger.ts`, `generate-canonical-ledger.ts`.
- **Reconciliation Engine**: `reconcile-rollforward.ts`, `reconcile-pairing-engine.ts`, `reconcile-non-asset-transactions.ts`, `reconcile-bank-pool.ts`, `apply-non-asset-reconciliation.ts`.
- **Integrity Gates & Audits**: `build-acceptance-gate.ts`, `run-vault-completion-gate.ts`, `audit-statement-trace-variance.ts`, `verify-input-manifest.ts`, `verify-source-of-truth-map.ts`.
- **Investment Analysis**: `validate-individual-stock-quantities.ts`, `vti-counterfactual-analysis.ts`, `rebuild-taxable-investments.ts`, `backfill-cost-basis.ts`.
- **Reporting & Wealth Walk**: `generate-wealth-walk.ts`, `generate-family-capital-plan.ts`, `generate-gap-report.ts`, `individual-stock-report.ts`.
- **Statement Parsers**: `r4-bank-residual-corrected.py`, `merge-betterment-evidence.py`, `parse-ally-statements.py`, `patch-pdf-classification.py`.
- **Overnight Runners**: `scripts/overnight/run_overnight.sh` and `scripts/overnight/morning_validate.sh`.

### 5. Forensics & Documentation (`docs/`)
- Forensic audit reports: `CHRONOLOGICAL_RECONCILIATION_AUDIT.md`, `DIVORCE_TRACING_REPORT_SPEC.md`, `DIVORCE_TRACING_WIREFRAME.html`, `FINANCIAL_CORPUS_INDEX.md`, `RECONCILIATION_GAPS.md`.
- Technical specs and plans: `docs/superpowers/plans/` and `docs/superpowers/specs/` covering tax-aware feasibility models.
- Tool review research: `docs/research/2026-08-22-retirement-planning-tool-review.md`.

### 6. Archived Code & Migrations (`archive/`, `migrations_archive/`, `analysis_scripts/`)
- **`archive/float-source/`**: Historical source snapshot from `float` split, including legacy frontend calculator (`frontend/retirement-calculator/retire/engine.js`), Gmail MCP server (`python/gmail_mcp/` using `fastmcp`), and source manifest scripts.
- **`migrations_archive/`**: Legacy database schema migration SQL scripts (`000_baseline.sql` through `024_remap_mint_categories.sql`).
- **`analysis_scripts/divorce_legacy/`**: Legacy divorce tracing Python scripts.

## Canonical entry points

- **Retirement Feasibility Report Generator**:
  - `python -m omar.planning.retirement_feasibility.render_report` — Executable script that compiles JSON scenario inputs into HTML, Markdown, and JSON retirement plan artifacts.
- **Retirement Engine Test Suite**:
  - `pytest omar/planning/retirement_feasibility/tests/` — Runs unit test suite for cash flows, tax computations, scenarios, and Monte Carlo engines.
- **Canonical Ledger Publisher**:
  - `npx tsx scripts/publish-accepted-ledger.ts` — Compiles and outputs accepted canonical ledger files to `canonical/`.
- **Acceptance & Integrity Gates**:
  - `npx tsx scripts/build-acceptance-gate.ts` — Evaluates balance observations and accounting leg balances against acceptance constraints.
  - `npx tsx scripts/run-vault-completion-gate.ts` — Executes vault completion verification checks.
- **Statement & Rollforward Reconciliation**:
  - `npx tsx scripts/reconcile-rollforward.ts` — Runs rollforward statement trace reconciliation.
  - `npx tsx scripts/validate-individual-stock-quantities.ts` — Audits individual stock position quantities against source statements.
- **Wealth Walk Generator**:
  - `npx tsx scripts/generate-wealth-walk.ts` — Computes historical net-worth trajectories across account masters.
- **Statement Indexing Pipeline**:
  - `python scripts/statement_index/discover.py` — Discovers and indexes financial statement documents in `omar/input/`.
  - `python scripts/statement_index/classify.py` — Classifies indexed financial statement documents.
- **Overnight Validation**:
  - `bash scripts/overnight/morning_validate.sh` — Runs integrity checks across financial indexes and statement data.
