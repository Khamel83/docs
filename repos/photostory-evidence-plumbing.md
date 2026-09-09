# LLM-OVERVIEW — photostory-evidence-plumbing
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
`photostory-evidence-plumbing` is a local, read-only evidence catalog over a single person's Apple Photos library and secondary evidence sources (such as Maya Gmail, iMessage, and Calendar). It establishes an immutable source ledger, derives neutral spatial and temporal moments, and projects visits, places, and occasions only where backed by photographed or recorded evidence.

Core non-negotiables:
- **Asset-first ledger**: Every enumerated item receives an immutable `asset_id`. Assets are never deleted; state transitions move `active → source_missing → tombstoned` (and reverse). Foreign keys enforce `ON DELETE RESTRICT` (no `CASCADE`).
- **Three distinct truth layers**: Source (immutable ledger/snapshots) → Derived (replaceable rule/model projections) → User (authoritative corrections). Derived data never overwrites source; model outputs never supersede user corrections.
- **Neutral moments**: Technical spatial/temporal groupings without sentiment or event claims; a single asset or a burst of receipts is a valid moment.
- **Evidence-backed visits**: Visits require photographed evidence associated with a place; continuous tracking and unphotographed arrival/departure inferences are prohibited.
- **Append-only corrections**: User corrections use typed anchors and survive rescans, model upgrades, and derived rebuilds.
- **Exclusion preservation**: Excluded assets remain in the ledger, Coverage metrics, and explicit search.
- **Local & loopback by default**: Network policy defaults to `model_policy = local_only`, remote payloads `off`, and HTTP host strictly loopback (`127.0.0.1:8787`).
- **Explicit uncertainty**: Material classifications expose evidence references, precision, and confidence scores.

## Machine & Host Ownership
No multi-host information is available; photostory-evidence-plumbing runs locally on the user's Mac (loopback `127.0.0.1:8787`).

## What is actually built
The repository consists of a Python 3.10+ package (`src/photostory`), a Swift PhotoKit bridge (`bridge/PhotoStoryBridge`), and 8 SQL database migrations (`migrations/`).

- **Core Infrastructure & Dependencies (`pyproject.toml`)**: Built with hatchling (`>=1.25`). Runtime dependencies are `click` (`>=8.1,<9`), `fastapi` (`>=0.115,<1`), and `uvicorn` (`>=0.30,<1`); dev dependencies are `httpx` and `pytest`.
- **Database & Storage Layer (`src/photostory/db/`)**: SQLite database at `$PHOTOSTORY_STATE_DIR/photostory.sqlite3` (default root `/Volumes/2TB_SSD/PhotoStory`), managed via 8 SQL migrations (`0001_ledger.sql` through `0008_correction_requests.sql`) owned directly in SQL without ORMs.
  - *Ledger Tables*: `libraries`, `entities`, `assets`, `asset_revisions`, `asset_metadata_snapshots`.
  - *Projection Tables*: `projection_generations`, `current_projection_generations`, `places`, `moments`, `moment_members`, `visits`, `claims`, `claim_supports`, `stories`, `story_sections`, `story_assets`.
  - *Corrections & Anchors*: `correction_commands`, `correction_state_events`, `correction_bindings`, `correction_anchor_manifests`, `correction_requests`.
  - *Job Queue*: `jobs` (resumable priority queue).
  - *Runtime Projections*: `setup_events`, `semantic_observations`.
  - *Multi-Source Evidence Plumbing (ADR-002)*: `source_registrations`, `source_runs`, `source_items`, `source_revisions`, `field_observations`, `derivation_runs`, `statements`, `mapping_entries`, `search_receipts`, `chronology_generations`, `current_chronology_generations`, `occasions`, `occasion_members`, `occasion_relationships`, `association_candidates`, `resolutions`, `statement_moment_bindings`, `occurrences`, `network_audit_events`, `statements_fts` (FTS5).
- **Swift PhotoKit Bridge (`bridge/PhotoStoryBridge/`)**: Compiled via `package-app.sh` into `PhotoStoryBridge.app` for stable macOS TCC permission identity (`Sources/Bridge/main.swift`, `Package.swift`, `Info.plist`). Communicates via JSON IPC over Launch Services or Subprocess transport for `doctor`, `authorize`, and asset enumeration.
- **Ingestion & Reconciliation (`src/photostory/ingest/`)**:
  - `Scanner` (`scanner.py`): Enumerates Photos assets via `BridgePhotosSource` (`bridge.py`), calculates deterministic source correlation HMACs using Keychain secrets, and reconciles records into the SQLite ledger (`reconcile.py`, `normalization.py`).
  - `Maya Integration` (`maya_import.py`, `readers/maya.py`, `readers/transactions.py`, `source_registry.py`, `mappings.py`, `maya_issues.py`): Imports bounded secondary evidence (e.g., `maya.gmail`, `maya.imessage`) into `source_items`, `source_revisions`, and `statements`. Audits outbound HTTP requests in `network_audit.py`.
- **Catalog Subsystem (`src/photostory/catalog/`)**: Generates asset observations and OCR text via `LocalSemanticAnalyzer` (`semantic.py`, `observations.py`, `vision.py`, `renditions.py`, `video_policy.py`), persisting results into `semantic_observations`.
- **Projections Engine (`src/photostory/projections/`)**:
  - Derives moments, visits, places, occasions, claims, occurrences, and stories across `moments.py`, `visits.py`, `places.py`, `occasions.py`, `claims.py`, `anchors.py`, `corrections.py`, `generations.py`, `history.py`, `stories.py`, `associations.py`, and `occurrences.py`.
  - Manages atomic generation cutovers (`projection_generations`, `chronology_generations`). User correction commands trigger `rebuild_chronology` queue jobs.
- **Search Subsystem (`src/photostory/search/`)**: Offers FTS5 full-text indexing and query processing over catalog observations (`fts.py`, `query.py`) and multi-source evidence statements (`evidence.py`, `statements_fts`).
- **Security & Keychain (`src/photostory/security/`)**: `MacOSKeychainSecretStore` (`keychain.py`) manages write-once source correlation keys and Maya MPAT tokens stored in macOS Keychain (`photostory.maya/mpat`).
- **Job Queue & Scheduler (`src/photostory/jobs/`)**: SQLite-backed job queue (`queue.py`) and disk-space aware task scheduler (`scheduler.py`).
- **HTTP API & Server-Rendered Web UI (`src/photostory/api/`, `src/photostory/web/`)**: FastAPI application (`app.py`) enforcing loopback host headers (`127.0.0.1`, `localhost`, `::1`) and CSRF tokens (`/api/csrf`). Serves `/healthz`, `/api/status`, `/api/mutations/corrections`, `/api/assets/...`, and dependency-free HTML views for `/` and `/search` (`web/routes.py`).
- **Daemon Service Management (`src/photostory/service/`)**: `LaunchAgentManager` (`launchagent.py`) installs/uninstalls a user macOS LaunchAgent plist for background loopback service execution.

## Canonical entry points
- **CLI Commands (`photostory`)**: Entry point defined in `src/photostory/cli/main.py` and `commands.py` (`photostory = "photostory.cli.main:main"` in `pyproject.toml`).
  - `photostory init`: Initializes state directory and write-once Keychain secret reference.
  - `photostory doctor`: Reports local readiness and Swift bridge status without reading Photos content or printing secrets.
  - `photostory authorize`: Triggers macOS Photos TCC authorization prompt under packaged bridge app identity.
  - `photostory scan --confirm-system-library`: Executes full System Photo Library enumeration and ledger reconciliation.
  - `photostory catalog` / `photostory reanalyze`: Analyzes active assets and populates FTS search indexes.
  - `photostory rebuild-moments` / `photostory rebuild-places` / `photostory coverage`: Rebuilds moment/place projection generations and outputs coverage stats.
  - `photostory search "<query>"` / `photostory asset <asset_id>`: Queries catalog observations and asset metadata.
  - `photostory maya configure` / `photostory maya probe` / `photostory maya import`: Stores Maya MPAT in Keychain, probes coverage views, and imports external evidence pages.
  - `photostory sources` / `photostory evidence-search "<query>"` / `photostory occurrence <id>`: Lists registered sources, searches multi-source evidence statements, and inspects occurrences.
  - `photostory correction --anchor-file <file>`: Ingests an append-only correction anchor file and enqueues a chronology rebuild job.
  - `photostory network-audit` / `photostory privacy-audit`: Displays active network policy limits and privacy boundaries.
  - `photostory migrate [--check]`: Applies database schema migrations and checks integrity.
  - `photostory serve`: Launches Uvicorn loopback HTTP service on `http://127.0.0.1:8787/`.
  - `photostory install-service` / `photostory uninstall-service`: Manages background macOS LaunchAgent service.
- **Bridge App Packaging**: `sh bridge/PhotoStoryBridge/package-app.sh` (builds `PhotoStoryBridge.app`).
- **API Server Factory**: `photostory.api.app:create_app(settings)`.
- **Configuration File**: `config.toml` (loaded and validated via `photostory.config.Settings.load()`; state path governed by `PHOTOSTORY_STATE_DIR`).
- **Schema Migrations**: `migrations/0001_ledger.sql` through `migrations/0008_correction_requests.sql` (applied via `photostory.db.migrate.migrate()`).
