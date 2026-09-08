# LLM-OVERVIEW — float
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
Float is a self-hosted personal finance web application designed for comprehensive financial tracking and historical analysis. It consolidates depository accounts, credit cards, investment holdings, mortgage amortization schedules, credit card perks/benefits, cryptocurrency balances, 529 education accounts, and manual real estate/loan valuations into a unified dashboard and progressive web app (PWA).

- **Technology Stack**: Node 22 (`>=22 <23`) runtime with ES modules, TypeScript (`tsx`), an Express 4 backend server (`src/server`), a Vite-built React 18 frontend single-page application (`frontend/`), and a local SQLite database (`float.db`) accessed via `better-sqlite3` configured with Write-Ahead Logging (WAL) journal mode, synchronous normal, and strict foreign key constraints. Note: While `AGENTS.md` mentions Next.js in its header summary, the actual codebase is built with Express and Vite React.
- **Core Purpose & Ownership**: Designed to maintain complete user ownership and data privacy without subscription fees or third-party data monetization. Data ingestion combines automated banking API feeds (Plaid API for Ally, Chase, Amex, Capital One), semi-automated Playwright browser automation (ScholarShare 529), public blockchain queries (Etherscan ETH crypto balances), property valuation APIs (Redfin), manual CSV file imports (Fidelity brokerage and HSA position files), and LLM-assisted transaction auto-categorization (Claude Haiku via OpenRouter).
- **Separation Baseline**: Personal financial data, user-specific directories (`omar/`), historical canonical records, and hardcoded user references were stripped across historical separation phases (PRs #91–#96), resulting in a clean baseline schema migration (`000_baseline.sql`) and generalized pipeline infrastructure (`src/pipelines/*`).

## Machine & Host Ownership
No multi-host information is available from the status probe output or AGENTS.md.

## What is actually built

### 1. Backend Express Application (`src/server/`)
- **App Engine & Middleware (`app.ts`, `index.ts`, `startup.ts`)**: Initializes the Express server on port 3002. Enforces JSON body parsing, CORS policies, Helmet security headers (with CSP disabled for SPA serving), proxy trust for TLS termination, and Passport Google OAuth 2.0 / session-based authentication (`auth.ts`). Includes fail-closed startup flags for migration and background job safety.
- **Demo Mode Server (`demo.ts`)**: Mounts an unauthenticated `/demo` application endpoint bound to `float-demo.db`, serving fuzzed demo data created via SQLite backup streaming to prevent database lock corruption.
- **API Routers (`src/server/routes/`)**:
  - `transactions.ts`: Paginated and filterable transaction browser (`/api/transactions`) supporting merchant renaming, category overrides, transaction state switching (`PENDING_REVIEW` vs. `RESOLVED`), and dynamic adapter views (`user_transaction_ledger` active vs `accepted_transaction_adapter` candidate).
  - `accounts.ts`: Queries depository, credit card, investment, loan, crypto, and real estate account rosters (`/api/accounts`) along with metadata stored in `app_metadata`.
  - `flow.ts`: Calculates monthly income, expenses, cash flow trends, and net transfer filtering (`/api/flow`).
  - `investments.ts`: Merges current Fidelity lot-level positions (`fidelity_positions`), activity logs (`investment_ledger`), and historical OCR records (`combined_investment_ledger`) into a unified portfolio feed (`/api/investments`).
  - `net-worth.ts`: Serves historical net worth snapshots (`/api/net-worth`), asset/liability category stratification, and historical data points (`net_worth_snapshots`).
  - `spending.ts`: Computes monthly category spending totals, category drilldowns, and top merchant aggregates (`/api/spending`).
  - `perks.ts`, `cards.ts`, & `loyalty.ts`: Exposes credit card fee structures (`cards`), annual perk allowances and reset cycles (`card_perks_ledger`), card benefit research logs (`card_research_log`), and loyalty program balances (`loyalty_accounts`, `loyalty_balances`).
  - `recurring.ts` & `rules.ts`: Manages recurring expense pattern detection rules (`recurring_rules`), tolerance windows, and merchant match criteria.
  - `crypto.ts`: Manages Ethereum wallet balances (`crypto_balances`).
  - `mortgage.ts`: Tracks mortgage principal balances, interest rates, monthly payment splits, and loan terms (`manual_liabilities`).
  - `setup.ts`: Handles Plaid Link token exchange (`plaid_items`) and OAuth callback redirection (`/setup/oauth-return`).
  - `inbox-memos.ts`: Delivers system alerts, perk expiration reminders, and credit card benefit change notifications (`inbox_memos`).
  - `settings.ts`, `balance.ts`, `ingress.ts`, & `fidelity.ts`: Manage system settings (`app_metadata`), account balance observations (`balance_observations`), raw payload ingress (`raw_ingress_ledger`), and Fidelity CSV imports.

### 2. Frontend SPA (`frontend/`)
- Built with Vite, React 18, React Router DOM, TanStack Query (`@tanstack/react-query`), and TailwindCSS.
- Dual layout engines supporting mobile PWA presentation (`PhoneShell.tsx`) and responsive widescreen displays (`DesktopShell.tsx`).
- **Screen Modules (`frontend/src/screens/`)**:
  - `HomeScreen.tsx`: Primary dashboard overview showing high-level cash balance, spending summaries, and alerts.
  - `CashFlowScreen.tsx`: Monthly income versus expense charts and cash flow trends.
  - `TransactionsScreen.tsx` & `TxnBrowser.tsx`: Full-featured transaction table with multi-field filtering, search, category assignment, and manual transfer pairing.
  - `SpendingScreen.tsx`: Visual spending breakdown by category, monthly trendlines, and category indicator dots (`CatDot.tsx`).
  - `AccountsScreen.tsx` & `LinkedAccountsScreen.tsx`: Grouped financial account list, Plaid link health status, and manual balance controls.
  - `NetWorthScreen.tsx` & `NWChart.tsx`: Area charts for historical net worth progression, asset class breakdowns, and sparklines (`Sparkline.tsx`).
  - `InvestmentsScreen.tsx` & `Invest.tsx`: Investment portfolio asset allocation, position breakdowns, and account holding details.
  - `MortgageScreen.tsx` & `MortgageView.tsx`: Mortgage balance payoff schedule and payment breakdown.
  - `PerksScreen.tsx` & `PerksList.tsx`: Credit card perk credit trackers, annual fee offset values, and reset date indicators.
  - `InboxScreen.tsx`: Notification log for perk changes, spend triggers, and action items.
  - `RecurringScreen.tsx` & `RulesScreen.tsx`: Active recurring subscriptions, match tolerances, and merchant normalization mapping rules.
  - `SettingsScreen.tsx`: App-wide parameters, account visibility toggles, and light/dark theme switcher (`useTheme.tsx`).

### 3. Database Architecture & Schema Baseline (`src/db/`)
- Database client setup in `src/db/client.ts` driving migrations in `src/db/migrations/`.
- Rebuilt clean baseline schema (`000_baseline.sql`) and incremental migrations (`001_initial_schema.sql` through `084_planning_asset_source_correction.sql`).
- **Primary Database Tables & Views**:
  - `user_transaction_ledger`: Primary transaction record (`transaction_id`, `date`, `merchant_name`, `amount_cents`, `category`, `status`, `account_token`, `transfer_pair_id`, `recurring_rule_id`, `is_exceptional`, `is_category_override`).
  - `cleansed_lookup_map`: Normalized merchant naming rules and autonomous AI categorization lookups.
  - `raw_ingress_ledger`: Raw JSON payload storage for Plaid, scrapers, and external ingress.
  - `plaid_items` & `plaid_cursors`: Active Plaid access tokens, institution identities, and API sync cursors.
  - `fidelity_accounts`, `fidelity_positions`, & `investment_ledger`: Stored investment accounts, lot snapshots, and activity logs.
  - `combined_investment_ledger` & `v_investment_activity`: Unified view aggregating historical Betterment OCR data and Fidelity investment records.
  - `crypto_balances`: Snapshot store for cryptocurrency token holdings, USD valuations, and wallet addresses.
  - `manual_assets` & `manual_liabilities`: Valuations for real estate, 529 education funds, and mortgage terms.
  - `net_worth_snapshots` & `v_net_worth`: Stratified historical net worth entries and aggregated views.
  - `recurring_rules`: Parametric rules matching recurring subscription expenses.
  - `cards`, `card_benefits`, `card_perks_ledger`, `benefit_value_history`, & `card_research_log`: Credit card metadata, perk values, remaining annual allowances, value revision histories, and AI research audit logs.
  - `inbox_memos`: Markdown-formatted system notifications, benefit updates, and spend alerts.
  - `loyalty_accounts` & `loyalty_balances`: Loyalty reward program balances and card linkages.
  - `app_metadata`: Key-value application state store.

### 4. Data Processing Pipelines (`src/pipelines/`)
- `canonical-records/`: Transaction verifiers (`verifier.ts`), transaction model specifications (`transaction.ts`), catalog management (`catalog.ts`), secure snapshot generators (`secure-source-snapshot.ts`), and vault scanners.
- `document-index/`: Multi-layer indexing engine (`layers.ts`), structured fact extraction (`facts.ts`), tombstone record tracking (`tombstones.ts`), decision ledgers (`decision-ledger.ts`), conflict resolution handlers (`conflict-resolution.ts`), and hash caching (`hash-cache.ts`).
- `non-asset-reconciliation/`: Custom statement parsers for major banking institutions (Ally, Amex, Capital One, Chase in `statement-parsers/`), transfer pair matcher (`transfers.ts`), expense rollups (`expenses.ts`), rollforward reports (`rollforward.ts`), and CSV integrity verifiers (`csv-safety.ts`).
- `source-index/`: Source document cataloging (`source-files.ts`), statement section extractor (`statement-sections.ts`), and content verification (`content.ts`).
- `statement-trace/`: Rigorous financial proof verification subsystem (`proof-verifier.ts`, `proof-ledger.ts`, `monthly-proof.ts`, `theory-comparison.ts`, `residual-diagnostic.ts`), statement loaders for Fidelity (`fidelity-loader.ts`) and DAF (`daf-loader.ts`), and Betterment parser (`betterment-parser.ts`).
- `source-inventory/divorce-pdf/`: Specialized PDF parser, reconciler, renderer, and publisher for financial agreement documents (`parse.ts`, `reconcile.ts`, `render.ts`, `publication.ts`).

### 5. Data Ingestion & Utility Scripts (`scripts/` & `python/`)
- **Ingestion & Reconciliation**:
  - `scripts/import-historical-xlsx.py` & `scripts/import-mint-extract.py`: Python ETL tools converting legacy spreadsheets and Mint CSV exports into SQLite transactions.
  - `scripts/generate-canonical-records.ts` & `scripts/build-canonical-history-db.ts`: Synthesizes raw observations into canonical transaction ledgers.
  - `scripts/import-fidelity-positions.ts` & `scripts/import-fidelity.ts`: Ingests Fidelity position export files (`Portfolio_Positions_*.csv`) and transaction history.
  - `scripts/import-combined-investments.ts`: Merges investment transaction histories across multiple accounts.
  - `scripts/fetch-crypto-balances.ts`: Queries public Ethereum JSON-RPC / Etherscan endpoints for wallet ETH balances.
  - `scripts/fetch-redfin-estimate.ts`: Fetches automated property valuation estimates for real estate assets.
  - `scripts/fetch-scholarshare.ts`: Playwright headless browser script scraping 529 account balances.
  - `scripts/auto-categorize.ts`: OpenRouter / Claude Haiku automated transaction categorization populating `cleansed_lookup_map`.
  - `scripts/detect-recurring.ts` & `scripts/infer-recurring-transactions.ts`: Evaluates transaction cadences to automatically identify recurring subscription payments.
  - `scripts/synthesize-transfer-pairs.ts`: Detects and links matching inter-account transfer pairs.
  - `scripts/snapshot-net-worth.ts`: Calculates current asset/liability totals and writes periodic entries to `net_worth_snapshots`.
  - `scripts/seed-demo.ts`: Produces an anonymized demo database (`float-demo.db`) via SQLite online backup.
  - `python/gmail_mcp/`: Python Model Context Protocol server (`server.py`, `auth.py`) extracting statement alerts and loyalty balances from Gmail.

## Canonical entry points
- **Development Server**: `npm run dev` — Launches `tsx watch src/server/index.ts`, running the Express server with live reload and Vite static SPA middleware on port 3002.
- **Production Build**: `npm run build` — Executes Node version preflight checks (`scripts/check-node-version.mjs`), compiles TypeScript (`tsc -p tsconfig.json`), and builds Vite frontend assets into `frontend/dist`.
- **Database Migrations**: `npm run migrate` — Runs `src/db/client.ts`, executing pending SQL migrations in `src/db/migrations/` against `float.db`.
- **Test Suite**: `npm test` — Validates Node version runtime and executes the Vitest test runner across `src/` and `scripts/`.
- **Full Historical Reconciliation Pipeline**: `npm run reconcile:full` — Chains historical XLSX import, Mint extract import, transfer pair synthesis, recurring transaction inference, Azlo account reconstruction, timeline reconciliation, and coverage report generation.
- **Investment Position Import**: `npm run import:positions` — Ingests dropped Fidelity CSV position files into `fidelity_positions`.
- **Demo Database Generator**: `npm run seed:demo` — Runs `scripts/seed-demo.ts` to output a fresh, anonymized `float-demo.db`.
- **Separation Verification**: `npm run check:pr-separation` — Ensures codebase isolation from personal data files.
- **CLI Agent Contracts**:
  - `agy`: Antigravity CLI contract using `-p "..."` / `--print="..."`, explicit timeout units (e.g. `600s`), mode flags (`--mode plan|code`), and supported effort flags (`low|medium|high`).
  - `g2k`: Gateway2000 OMP client wrapper (`g2k -p "..."`, `g2k-bg`, `g2k-sensitive`, `g2k-check`) routing prompts through `https://gateway.khamel.com`.
