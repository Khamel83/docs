# LLM-OVERVIEW — baywatch
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
Baywatch (`baywatch` package, v0.2.4) is Homelab's read-only observer application written in Python (3.11+). It operates as a stateless one-shot monitoring process invoked on a schedule (e.g., via `baywatch.timer` every five minutes).
- **Core Functionality**: Queries Homelab's API (`GET /projects`, `GET /projects/{project_id}`) for normalized project declarations, evaluates deterministic health checks on active (`observing`, `managed`) projects, queues observation events, and posts check states back to Homelab (`POST /observations`). Quiet projects (`standby`, `archived`) do not receive detail queries or check evaluations.
- **Architectural Boundary**: Baywatch is strictly read-only. Homelab owns project registration, catalog management, secrets, notifications, and repair execution. Baywatch never scans networks or machines, registers projects, executes repairs, decrypts secrets, talks directly to Maya, or accepts Docker/SSH/shell execution authority.
- **State & Outbox**: Stores atomic observation events locally (`observations/`), maintains a deduplication cursor (`observation-dedup.json`), persists an active outbox (`outbox/`), and caches catalog views (`project-catalog-cache.json`) for offline probe execution during API outages.
- **Exit Code Contract**:
  - `0`: Completed without incidents (all checks healthy).
  - `3`: Diagnostic success with incidents (one or more checks `failed` or `unknown`), accepted by systemd as non-failing completion.
  - `1`: Runtime execution failure, transport outage, or state persistence error.
  - `2`: Invalid CLI configuration or command arguments.
  - Missing, stale, queued, or unknown evidence is never rendered as healthy.

## Machine & Host Ownership
No multi-host or host execution information is available in the status probe output or `AGENTS.md`.

## What is actually built
The repository consists of 14 core Python modules under `src/baywatch/`, standard JSON contracts in `schemas/`, test suites in `tests/`, and reference deployment units in `deploy/`.
- **CLI & Configuration (`src/baywatch/cli.py`)**: Console entrypoint `baywatch` handling flags (`--base-url`, `--state-dir`, `--no-llm`, `--fixture-dir`, `--timeout`, `--run-budget`) and environment variables (`BAYWATCH_HOMELAB_BASE_URL`, `BAYWATCH_STATE_DIR`, `BAYWATCH_HOMELAB_API_TOKEN`). Maps internal exceptions to process exit codes.
- **Runner & Orchestrator (`src/baywatch/runner.py`)**: Acquires an exclusive file lock (`.lock`), coordinates catalog ingestion, runs check probes sequentially within time budgets, triggers optional LLM diagnosis on failure/unknown checks, persists observations, delivers batches to Homelab, and generates run reports.
- **Domain Models (`src/baywatch/models.py`)**: Typed dataclasses for `CheckSpec`, `ProjectView`, `Observation`, `ProviderStatus`, and `RunReport`. Computes state fingerprints for change detection. (`Incident` remains as unconsumed legacy model residue).
- **Transport & Homelab Client (`src/baywatch/transport.py`, `src/baywatch/homelab.py`)**: Abstraction over HTTP/JSON requests with header token authentication (`X-API-Key`) and local fixture replay (`FixtureTransport`). Homelab client handles `/projects`, `/projects/{project_id}`, `/observations`, and `/evidence/{project}/{check}` endpoints.
- **Check Adapter Probes (`src/baywatch/checks.py`)**: Sequential check executors:
  - `self`: Returns local runner heartbeat.
  - `http`: Probes endpoint status code and response latency.
  - `tcp`: Verifies socket connectability and latency.
  - `systemd`: Runs fixed `/usr/bin/systemctl show <target>` argv without shell invocation.
  - `docker-health`: Fetches delegated container evidence from Homelab API (no local Docker socket access).
- **Observation Store & Catalog Cache (`src/baywatch/observations.py`, `src/baywatch/catalog_cache.py`)**: Atomic file storage for immutable observation events, outbox delivery queueing, deduplication cursors, and local caching of active project catalogs.
- **LLM Diagnosis & Proxy (`src/baywatch/providers.py`, `src/baywatch/ollama_host_proxy.py`)**: Provides optional advisory text diagnosis for failing/unknown checks via Ollama (`ornith` model) with OpenRouter fallback. Includes `ollama_host_proxy.py`, a loopback-only proxy (`127.0.0.1:11435` -> `11434`) enforcing path filtering (`/api/tags`, `/api/generate`), 1 MiB body limits, and 60-second timeouts. Model outputs are strictly advisory and cannot alter core evidence, exit codes, or trigger repairs.
- **Sanitization, Reporting & Retention (`src/baywatch/sanitize.py`, `src/baywatch/report.py`, `src/baywatch/retention.py`)**: Masks sensitive text/keys before state persistence or report generation. Generates Markdown and JSON run records, retaining the last 288 runs up to 7 days.
- **Contracts & Schemas (`schemas/*.json`, `homelab.yaml`)**: Published JSON schemas (`project.schema.json`, `check.schema.json`, `observation.schema.json`) defining API payload contracts, aligned with `homelab.yaml`.
- **Retired History**:
  - OCI-backed AI review workflow is retired (`8e105bc`).
  - Legacy shell pipeline `triage.sh.example` (Hestia/Argus/OpenRouter/Maya shell script) is uncalled, legacy history that violates current read-only and no-Maya execution constraints.

## Canonical entry points
- **Production CLI Entrypoint**:
  ```bash
  baywatch --base-url <homelab-url> --state-dir /var/lib/baywatch
  ```
- **Module Execution**:
  ```bash
  python3 -m baywatch.cli --base-url <homelab-url> --state-dir /var/lib/baywatch
  ```
- **Local Fixture Diagnostic (Offline / Test Mode)**:
  ```bash
  PYTHONPATH=src python3 -m baywatch.cli \
    --fixture-dir tests/fixtures/homelab-v1 \
    --state-dir ./reports \
    --no-llm
  ```
- **Loopback Ollama Proxy Execution**:
  ```bash
  python3 -m baywatch.ollama_host_proxy
  ```
- **Source Verification & Quality Gates**:
  ```bash
  git status --short --branch
  python3 -W error::ResourceWarning -m unittest discover -s tests -t . -v
  python3 -m pytest -q -p no:cacheprovider
  python3 -m compileall -q src tests
  ruff check src tests
  ruff format --check src tests
  ruff check src --select F,B,S
  ```
- **Deployment Unit Examples**:
  - `deploy/baywatch.service.example` (Systemd oneshot service template)
  - `deploy/baywatch.timer.example` (Systemd 5-minute schedule timer template)
  - `deploy/com.baywatch.ollama-host-proxy.plist` (macOS LaunchAgent template for local Ollama proxy)
