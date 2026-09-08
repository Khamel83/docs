# LLM-OVERVIEW — argus-ops
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
`argus-ops` is a private operations repository storing operational records, architectural decision records (ADRs), audit reports, production stabilization plans, design specifications, and bounded validation script packages for the Argus system (`Khamel83/argus`). It does not contain the main Argus application source code. Instead, it provides execution evidence, audited release snapshots (such as September 1 and September 7, 2026 restoration audits), provider receipt matrices, status snapshots, and remote validation canaries.

## Machine & Host Ownership
Based on code configurations and validation scripts (`audits/2026-09-07/`):
- **Homelab Host**: Host machine running local app data paths (e.g. `/mnt/fast-storage/appdata/argus-validation/20260907` and `/Volumes/2TB_SSD/GitHub/homelab`), accessible via Tailscale host `homelab.deer-panga.ts.net`.
- **Docker Containers**:
  - `argus`: Main application container running the REST API on `http://127.0.0.1:8000`. Validated against image digest `sha256:0536b56458b6b64592c6625a326d14a1d895e5561cdb10334788be875ac78812` and source revision `8c9dad1356517cd01c714da84401aaed9242cc54`.
  - `argus-mcp`: Model Context Protocol service container running on `http://127.0.0.1:8001`.
- **Database**: PostgreSQL instance accessed via SQLAlchemy connection string `ARGUS_DB_URL`.

## What is actually built

### 1. Bounded Audit & Validation Runner Suite (`audits/2026-09-07/`)
- `run_one_remote.py`: Homelab at-most-once validation launcher. Checks container health (`docker inspect argus` / `argus-mcp`), verifies exact image digest (`sha256:0536...`), manages execution markers in `/mnt/fast-storage/appdata/argus-validation/20260907`, and executes authorized canary operations (`serper`, `valyu`, `extraction`, `mcp`).
- `authority_snapshot.py`: Queries `/app/runtime-manifest.json`, checks `/api/admin/status` with `X-Admin-API-Key` (and tests unauthenticated admin rejection), and reads PostgreSQL tables using SQLAlchemy (`ARGUS_DB_URL`).
- `bounded_provider_canary.py`: Executes in-container provider verification against authorized search providers (`serper`, `valyu`), verifying runtime revision `8c9dad1356517cd01c714da84401aaed9242cc54` and auditing PostgreSQL ledger entries.
- `extraction_maya_canary.py`: Performs a single authorized content extraction via `POST /api/v2/extract` (target: PEP 257 page) using bearer credentials loaded from `ARGUS_CALLER_CREDENTIALS_JSON`.
- `mcp_authority_gate.py` & `external_mcp_gate.py`: Validates authenticated MCP discovery and read-only HTTP authority endpoints over internal HTTP (`http://127.0.0.1:8001/mcp`) and external Tailscale endpoints (`https://homelab.deer-panga.ts.net/api/admin/status`), decrypting local vault credentials (`secrets decrypt`).

### 2. Execution Audit Records (`audits/`)
- `audits/2026-09-07/`: Authoritative restoration audit package including `AUDIT_REPORT.md`, `PROVIDER_MATRIX.md` (provider evidence), `PROGRESS.md` (checkpoints), and validation scripts.
- `audits/2026-09-01/`: Historical baseline audit snapshot including `AUDIT_REPORT.md`, `SOURCE_MANIFEST.json`, `SYSTEM_TOPOLOGY.md`, and `HEALTH_CHECK_RUNBOOK.md`.

### 3. Architecture Decisions & Production Plans (`adrs/`, `designs/`, `plans/`)
- `adrs/`: ADR 0007 (Guarded Acquisition Boundary), ADR 0008 (Authoritative Persistence and Recovery), and proposed amendments for ADRs 0004–0006.
- `designs/`: `2026-09-02-argus-production-stabilization-design.md` (system architecture stabilization).
- `plans/`: Implementation blueprints for public-private repo split (`2026-09-01-public-private-split.md`), production stabilization, authoritative persistence/recovery, guarded acquisition with SearXNG, and spend workflow evidence release gates.
- `PUBLICATION.md`: Documented verification boundary for public audit publication and merge integrity.

## Canonical entry points
- **Operational & Audit Index**: `README.md`.
- **Public/Private Split Contract**: `PUBLICATION.md`.
- **Validation Launcher Script**: `audits/2026-09-07/run_one_remote.py`.
- **System Authority Inspector**: `audits/2026-09-07/authority_snapshot.py`.
- **Canary Validation Execution Scripts**:
  - Provider Canary: `audits/2026-09-07/bounded_provider_canary.py`
  - Content Extraction Canary: `audits/2026-09-07/extraction_maya_canary.py`
  - MCP Authority Gate: `audits/2026-09-07/mcp_authority_gate.py` & `external_mcp_gate.py`
- **Architectural Specifications**: `adrs/`, `designs/2026-09-02-argus-production-stabilization-design.md`, and `plans/`.
