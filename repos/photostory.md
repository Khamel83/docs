# LLM-OVERVIEW — photostory
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is

PhotoStory is a local, read-only evidence catalog over one person's Apple Photos library. It enumerates raw assets, records an immutable source ledger, generates semantic descriptions, sessionizes assets into neutral moments, and derives visits, places, and occasions only when backed by photographed evidence.

Core invariants and non-negotiables:
- **Read-only Apple Photos source:** PhotoStory never writes, edits, deletes, or reorganizes Photos library assets.
- **Asset-first durability:** Every asset receives a permanent `asset_id`. Status transitions move `active → source_missing → tombstoned` (and back). Foreign keys strictly enforce `ON DELETE RESTRICT`.
- **Three distinct truth layers:** Immutable source snapshots → replaceable derived rule/model projections → authoritative user corrections. Derived output never overwrites source; later model output never supersedes a user correction.
- **Neutral moments & evidence-backed visits:** Moments are technical time/space asset clusters. Visits require photographed evidence at a place—never tracking presence or unphotographed activity.
- **Local-first privacy defaults:** Network model policy defaults to `local_only`; visual and text remote payload axes default to `off`. Server host binds strictly to loopback (`127.0.0.1`).
- **Unimplemented historical design:** ADR-002 (multi-source reconstruction for calendar/messages/transactions) exists as a decision record and unintegrated holding prototype (`docs/holding/reconstruction/reconstruct.py`), but is **not** implemented in active runtime code.

## Machine & Host Ownership

There is no multi-host information available. PhotoStory runs entirely on a single local macOS machine (Apple Silicon Mac with local Apple Photos library access). Host validation (`Settings.validate()` and FastAPI middleware) strictly rejects non-loopback bindings outside `127.0.0.1`, `::1`, and `localhost`.

## What is actually built

### 1. Ingest & Source Ledger (`src/photostory/ingest/`, `src/photostory/db/repositories/assets.py`)
- **`BridgeClient` & Transports (`ingest/bridge.py`):** JSON-line IPC interface communicating with the Swift PhotoKit bridge. `LaunchServicesBridgeTransport` executes the helper via `open -a` to run under the bridge's dedicated macOS TCC app bundle identity (`com.photostory.bridge`); `SubprocessBridgeTransport` acts as fallback.
- **`Scanner` (`ingest/scanner.py`):** Iterates bridge assets, passes raw dictionary metadata through `normalize_asset` (`ingest/normalization.py`), computes deterministic HMAC correlation keys using the Keychain secret, and records observations in the database.
- **Source Reconciliation (`ingest/reconcile.py`):** `next_absence_state()` manages `active → source_missing → tombstoned` state transitions requiring a 10-minute minimum confirmation window on healthy scans.
- **Database Schema (`migrations/0001_ledger.sql`):** Tables include `libraries`, `entities`, `assets`, `asset_revisions`, `asset_identifiers`, `asset_metadata_snapshots`, `asset_resources`, `asset_source_state_events`, and `scan_checkpoints`.

### 2. Swift PhotoKit Bridge (`bridge/PhotoStoryBridge/`)
- Swift package executable (`Sources/Bridge/main.swift`, `Package.swift`, `Info.plist`) providing read-only access to `PHPhotoLibrary` and `PHAsset`.
- Supported IPC operations: `doctor` (reports OS version and PhotoKit authorization), `authorize` (requests macOS Photos permission), `enumerate` (fetches user assets with hidden/burst options), and `observe_changes` (`PHPhotoLibraryChangeObserver`).

### 3. Catalog & Vision Subsystem (`src/photostory/catalog/`)
- **`LocalSemanticAnalyzer` (`catalog/semantic.py`):** Evaluates renditions using local vision functions or deterministic fallback (`catalog/vision.py`). Refused or malformed model output records explicit failure reasons (`model_refused`, `model_failed`) without silent network fallbacks.
- **Observation Tracking (`catalog/observations.py`):** `invalidated_stages` flags derived stages (`ocr`, `description`, `feature_print`, `presentation`) for re-analysis when render hashes change.
- **Database Schema (`migrations/0005_runtime_projections.sql`):** Stores results in `semantic_observations` (enforcing `UNIQUE(asset_id, input_hash)`) and setup events in `setup_events`.

### 4. Projections & Corrections (`src/photostory/projections/`)
- **`build_moments` (`projections/moments.py`):** Sessionizes active assets into neutral moments based on time thresholds and haversine spatial distance, handling A-X-A spatial anomaly checks. Generates deterministic UUIDv5 moment identifiers (`_moment_id`).
- **`ProjectionRepository` (`projections/generations.py`):** Manages non-destructive shadow generation builds (`build_moment_generation`) and atomic generation publication (`publish_generation`) using `projection_generations` and `current_projection_generations`.
- **Domain Projections & Corrections:** `places.py`, `visits.py`, `occasions.py`, `claims.py`, `stories.py`, `history.py`, `anchors.py`, and `corrections.py` handle spatial clustering, evidence claims, context links, and append-only user correction commands.
- **Database Schema (`migrations/0002_projections.sql`, `0003_corrections_and_anchors.sql`):** Tables include `projection_generations`, `current_projection_generations`, `places`, `moments`, `moment_assets`, `visits`, `correction_commands`, `correction_state_events`, `correction_bindings`, `claims`, and `context_links`.

### 5. Search & FTS Index (`src/photostory/search/`)
- **`SearchIndex` (`search/fts.py`):** Maintains SQLite FTS5 table `photostory_search` (`asset_id UNINDEXED, description, ocr_text, place, media_type`) using unicode61 tokenization.
- **`parse_query` (`search/query.py`):** Parses filter prefixes (`media:`, `place:`, `status:`, `year:`, `month:`, `date:`) and four-digit year tokens.

### 6. Jobs Queue & Scheduler (`src/photostory/jobs/`)
- **`JobQueue` (`jobs/queue.py`):** SQLite-backed priority queue operating on the `jobs` table (`migrations/0004_jobs.sql`). Manages statuses `queued`, `running`, `completed`, `failed`, and `superseded`.
- **`Scheduler` & `ChangeDrain` (`jobs/scheduler.py`):** Monitors thermal states and memory pressure to govern background execution lanes (`foreground`, `idle`). Coalesces observer notifications over 30-second windows.

### 7. HTTP API & Web UI (`src/photostory/api/`, `src/photostory/web/`)
- **FastAPI Loopback Server (`api/app.py`):** Enforces strict loopback host checks (`loopback_guard`) and injects CSP headers. Endpoints: `/healthz`, `/api/csrf`, `/api/status`, `/api/mutations/corrections` (requiring `X-CSRF-Token: local` and `Idempotency-Key`), and `/api/assets/{asset_id}/resource/{resource_id}` (stub/404 protecting raw resource paths).
- **Local Web UI (`web/routes.py`):** Server-rendered HTML dashboard (`/`) displaying catalog statistics and interactive search interface (`/search`).

### 8. Security & Configuration (`src/photostory/security/`, `src/photostory/service/`, `src/photostory/config.py`)
- **`MacOSKeychainSecretStore` (`security/keychain.py`):** Interacts with `/usr/bin/security` to store and retrieve the write-once source correlation HMAC key at `com.photostory.installation/source_correlation_hmac`.
- **`LaunchAgentManager` (`service/launchagent.py`):** Manages user LaunchAgent lifecycle at `~/Library/LaunchAgents/com.photostory.agent.plist`.
- **`Settings` (`config.py`):** Loads TOML configuration from `config.toml` or `PHOTOSTORY_STATE_DIR` (default `/Volumes/2TB_SSD/PhotoStory`). Validates loopback binding and network payload rules.

## Canonical entry points

- **CLI Application:** `photostory` (registered in `pyproject.toml` to `photostory.cli.main:main`).
  - Core CLI commands: `init`, `doctor`, `migrate`, `scan` (requires `--confirm-system-library`), `authorize`, `catalog`, `reanalyze`, `rebuild-moments`, `rebuild-places`, `search`, `asset`, `coverage`, `serve`, `network-audit`, `privacy-audit`, `install-service`, `uninstall-service`.
- **Loopback Web Server:** `uvicorn` invocation of `photostory.api.app:create_app` via `photostory serve` (default: `http://127.0.0.1:8787/`).
- **Swift PhotoKit Helper:** `bridge/PhotoStoryBridge/Sources/Bridge/main.swift` (invoked via CLI subprocess or LaunchServices transport with `--doctor`, `--authorize`, `--enumerate`, or JSON-line stdin).
- **Database Migrations:** `photostory.db.migrate:migrate` (executes SQL scripts `0001_ledger.sql` through `0005_runtime_projections.sql` on `photostory.sqlite3`).
