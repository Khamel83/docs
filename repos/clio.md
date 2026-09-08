# LLM-OVERVIEW — clio
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
Clio is a self-hosted capture-and-dispatch control plane and the usability/documentation layer for the homelab `repo-fleet-docs` ecosystem. It provides a multi-channel intake gateway for notes, URLs, and files, classifying incoming drops, generating GitHub issues for actionable tasks, dispatching autonomous agent workers to resolve them via pull requests, and auditing those PRs with automated confidence scoring.

### Core Architecture & Responsibilities
- **Multi-Source Drop Intake**: Ingests raw drops via web form, REST API (`POST /api/intake`), iOS Shortcuts, WhatsApp, Hermes, or Maya ingest forwarding.
- **Pipeline Processing**: `run_pipeline()` fetches URL contents, applies heuristic and LLM classification, and routes items to GitHub issues, calendar events, or archives.
- **Chronos Dispatch Loop**: Background dispatcher polling every 5 minutes for actionable GitHub issues, spawning `oh-my-pi` (`omp`) workers to plan, execute code changes, and open PRs.
- **Automated Webhook Audit**: GitHub webhooks trigger immediate automated PR code reviews (`_review_pr()`), posting 0-100 confidence score comments, attaching audit labels, and updating Slack threads.
- **Fleet Documentation Mirroring**: Consumes homelab-generated repo overviews and mirrors them into `docs/repo-registry/` for phone-browsable access. Clio does **not** run fleet scanning, overview generation, or publishing.
- **System Philosophy**: Prioritizes human time by keeping interfaces concise and automating repetitive work; optimizes LLM cost by using value-priced completion gateways for classification and auditing while reserving frontier models for orchestrating complex agent plans; builds reactive, trigger-based flows over polling loops.

## Machine & Host Ownership
This repository runs on the macmini host (`/Users/macmini/releases/clio`); no multi-host deployment information is specified in `AGENTS.md` or status probe output.

## What is actually built

### 1. Intake & Classification Gateway
- **Ingestion Endpoint**: `POST /api/intake` receives unstructured drops from human users, iOS shortcuts, WhatsApp, and machine forwarders (Hermes, Maya ingest forwarding, iMessage/Gmail sync).
- **Classification Pipeline (`run_pipeline()`)**: Fetches linked URL content, executes heuristic classification rules, runs LLM classification calls, and assigns routing targets.
- **Route Hint Reconciliation**: `DeliveryWorkflow.route` is populated directly from `IntakeItem.route_hint` (reconciled in `reconciler.py`), ensuring non-actionable drops (notes, reminders, iMessage texts) do not trigger code dispatch pipelines or Slack activity noise.
- **Source Registry**: Maintains source metadata and seed records for channels including Gmail (`fix(intake): seed gmail source row`) and iMessage.

### 2. Fleet Role Contract & Strict API Isolation
- **Executor Contract**: Clio serves strictly as an execution worker when dispatched by Maya or Hermes. Clio **calls nobody and fetches nothing** outside its payload context.
- **Zero-Client Isolation Rule**: Codebase contains no HTTP clients directed at Maya or Hermes APIs (with the single permitted exception of watchdog health checks on Hermes `/health`).
- **Machine Authentication (`machine_auth.py`)**: Inbound machine requests authenticate using `CLIO_INTERNAL_TOKEN` (granting `internal` role); destructive and administrative operations remain admin-only.
- **Context Pack Ingestion**: Operational context is injected into intake payloads by callers (Hermes fetches ContextPacks from Maya's `GET /fleet/context/{slug}`).

### 3. Chronos Dispatcher & Multi-Lane Orchestration
- **Polling Dispatcher**: Chronos polls active GitHub issues every 5 minutes to select eligible tasks for agent execution.
- **Lane Allocation**: Supports multi-lane workflow routing, including Lane A (code execution), Lane B (research tasks), and Lane D (roadmap tracking).
- **Re-Ingest Loop Prevention**: Marks context-only sources (such as `lane_b_research`) to prevent recursive ingest loops during research task execution.
- **Reaper & Session Hardening**: Includes robust task reaper classification, preflight activity announcements, verified git push checks, and per-project database session isolation in `_index_repos` across watched repositories (`CLIO_WATCHED_REPOS`).

### 4. OMP Execution Worker & Gateway Integration
- **Agent Worker Orchestration**: Spawns `oh-my-pi` (`omp`) execution workers using a structured pipeline: cost-effective model planning -> review -> execution -> pull request submission.
- **Loopback Gateway & Webhook Recovery**: Maintains automated recovery routines for OMP loopback gateway connections and webhook delivery paths.

### 5. Webhook PR Review & Automated Audit Engine
- **Automated PR Reviewer (`_review_pr()`)**: Listens on `POST /api/github/webhook` to evaluate code changes, generate 0-100 confidence score comments, and attach audit labels upon PR creation or update.
- **Webhook Truth Verification**: Verifies incoming POST signatures and webhook payload integrity via `_ensure_webhooks`.
- **Single-Anchor Activity Threads (`activity_thread.py`)**: Posts per-issue lifecycle events to Slack (`#clio-activity`). Uses a two-phase commit pattern (claiming thread rows fast, invoking Slack APIs outside DB transactions) to prevent race conditions and duplicate top-level anchors.
- **Activity Noise Suppression**: Suppresses `#clio-activity` notification broadcasts for non-code drops and noise sources (`fix(delivery): suppress #clio-activity posts for noise sources`).

### 6. Job Scoreboard & Telemetry System
- **Scoreboard API (`/api/scoreboard`)**: Background-job telemetry system (#586) tracking execution history, task run metrics, job performance, and pipeline health.
- **Job Monitoring**: Monitors scheduled task execution across Chronos, intake processing, and webhook delivery.

### 7. Database Concurrency & Scheduler Resilience
- **SQLite Concurrency**: SQLite database configured with a 5-minute `busy_timeout` to mitigate database lock contention during concurrent worker dispatch and webhook handling.
- **Argus HTTP Isolation**: Argus HTTP calls are executed outside database sessions (`fix(scheduler): move Argus HTTP outside DB session`).
- **Scheduler Watchdog**: Watchdog process configured with a 60-second timeout and a threshold of 5 failures to ensure scheduler stability.

### 8. Label Taxonomy & Repository Synchronization
- **Canonical Label Taxonomy**: Rebuilt label structure documented in `docs/LABEL_TAXONOMY.md` (29 clio-specific labels).
- **Fleet Label Seeding**: `scripts/seed-github-labels.sh` synchronizes standardized label sets across all 10 watched repositories in `CLIO_WATCHED_REPOS`.

### 9. Documentation Mirroring & Repository Registry
- **Local Overviews**: Stores and mirrors homelab repo context packs in `docs/repo-registry/`.
- **Mobile Access**: Provides phone-browsable documentation access for fleet repository overviews.

## Canonical entry points
- `POST /api/intake` — Main intake gateway endpoint for notes, URLs, files, and forwarded drops.
- `POST /api/github/webhook` — GitHub webhook handler for automated PR review auditing (`_review_pr()`) and issue lifecycle events.
- `GET /api/scoreboard` — Telemetry API route for querying background jobs and worker task runs.
- `machine_auth.py` — Authentication module verifying `CLIO_INTERNAL_TOKEN` for machine callers.
- `reconciler.py` — Intake reconciliation module populating `DeliveryWorkflow.route` from `IntakeItem.route_hint`.
- `activity_thread.py` — Single-anchor Slack activity thread management module.
- `scripts/seed-github-labels.sh` — Management script for seeding unified label taxonomies across watched repos.
- `scripts/status.py` — Diagnostic status probe script (executed via `JANITOR_RUN_STATUS_PROBE=1`).
- `docs/AGENTS.md` — Primary constitution containing agent instructions, fleet contracts, and system vocabulary.
- `docs/CAPABILITIES.md` — Authoritative reference catalog of API routes, scheduler jobs, subpackages, and connectors.
- `docs/LABEL_TAXONOMY.md` — Canonical reference for GitHub label categories, colors, and issue management.
- `docs/HANDOFF.md` — Current operational status, dispatch state, known issues, and recent infrastructure updates.
