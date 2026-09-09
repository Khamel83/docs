# LLM-OVERVIEW — homelab-maya-signing-backup
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is

`homelab-maya-signing-backup` (homelab) is the infrastructure source of truth for a personal multi-machine environment. It manages production Docker container deployment, SOPS encrypted secrets management, durable system state, scheduled APFS/systemd backup automation—including dedicated Maya iMessage signing recovery key preservation—health monitoring via Baywatch, and normalized project catalog reconciliation across 63 registered projects.

## Machine & Host Ownership

Based on code configurations, `AGENTS.md`, and deployment tests (`test_placement_auditor.py`, `test_regen_tiers.py`):
- **Mac mini**: Primary development host and native runtime environment. Code edits, screensharing, and CLI invocations execute here (`hl` front-door CLI).
- **Homelab**: Primary production Docker host and durable infrastructure owner. Runs production containers (Tier 1/2/3), hosts `homelab-api`, stores health observations in SQLite, and executes systemd backup timers. Maps to the `production` host in `TIER_HOST_MAP`.
- **MBA / Work MBP**: Access clients for SSH and Tailscale management; not active runtime execution environments.
- **OCI**: Retained standby/deprecated host. Explicitly mapped to `deprecated` host status in service tier generation; legacy placement records are not live production truth.
*(Live status probe disabled — set `JANITOR_RUN_STATUS_PROBE=1` to execute `scripts/status.py`).*

## What is actually built

### 1. Tiered Container Architecture (`docker-compose.yml`, `Makefile`, `services/`)
Services are split into a three-tier update topology managed via Docker bridge network `homelab` (`172.20.0.0/16`):
- **Tier 1 (Critical)**: Core infrastructure (DNS, proxy, management). Updated manually (`make update-critical`).
- **Tier 2 (Stable)**: Auto-update safe applications (media, monitoring, internal tools).
- **Tier 3 (Experimental)**: Opt-in testing containers.
- **Service Modules**: Microservices in `services/` including `apprise`, `argus`, `atlas-api`, `atlas-edge-consumer`, `atlas-edge-worker`, `atlas-postgres`, `atlas-tunnel`, `audiobookshelf`, `authentik`, `autoheal`, and `baywatch`.

### 2. Baywatch Monitoring & Health Engine (`services/baywatch/`, `tests/test_baywatch_deployment.py`)
- **Baywatch Watcher**: Read-only `oneshot` systemd service (`baywatch.service`) scheduled every 5 minutes (`baywatch.timer`). Runs with `--no-llm` and strict privilege isolation (`User=baywatch`, `NoNewPrivileges=true`, empty `CapabilityBoundingSet`, lock file `/var/lib/baywatch/baywatch.lock`).
- **Incident Reporting**: Reads `/projects` and `/projects/{id}`, emitting sanitized observations to `homelab-api:/observations`. Returns exit code `3` when incidents are found (treated as a successful systemd diagnostic).
- **Homelab API Boundaries**: Public `GET /health` separates informational `services_stopped` from `services_unhealthy`. Exited containers do not degrade system health by default. Broad service mutation endpoints (`POST /services`, `DELETE /services/{name}`) are retired (HTTP 410).

### 3. Backup & Recovery Automation (`systemd-timers/`, `secrets/`, `scripts/`)
- **Maya iMessage Signing Recovery**: Preserves Maya iMessage signing recovery state via `secrets/maya-imessage-signing-recovery.tar.age` and `systemd-timers/backup-maya.timer`.
- **Systemd Timers**: Timers in `systemd-timers/` execute scheduled runs: `backup-main.timer`, `backup-main2.timer`, `backup-maya.timer`, `clamav-scan.timer`, and `czkawka-scan.timer`.
- **APFS Snapshots**: Utilizes `scripts/apfs-backup-resilient.sh` for resilient local storage backups.

### 4. Catalog Reconciliation & CLI Tooling (`hestia/`, `config/project-catalog.yml`, `scripts/homelab_cli.py`)
- **Normalized Catalog**: `config/project-catalog.yml` defines 63 personal projects synchronized 1:1 with Hestia registry (`hestia/registry.yml`).
- **Monitoring States**: Projects are classified as `managed`, `observing` (active Baywatch scope), `standby`, or `archived` (quiet).
- **Front Door CLI**: `scripts/homelab_cli.py` (`hl`) provides developer commands (`hl onboard`, `hl remote-status`). Reconciled via `make project-audit` and `make project-graph`.

### 5. Secrets Management (`secrets/`, `docs/SECRETS.md`)
- Encrypted environment state using SOPS and Age.
- Vault inventory includes `arb.env.encrypted`, `atlas.env.encrypted`, `backup-snapshot.env.encrypted`, `clio.env.encrypted`, `convex.env.encrypted`, `float.env.encrypted`, `homelab.env.encrypted`, `maya.env.encrypted`, `openclaw.env.encrypted`, and `maya-imessage-signing-recovery.tar.age`.

### 6. Orchestration & Intelligence (`AGENTS.md`, `config/`)
- **ONE_SHOT v14 Contract**: Governs AI agent orchestration via `dispatch-run`. Task routing spans intelligence tiers (`v4-flash`, `codex`, `gemini`, `mm2.7`, `mm2.5`).
- **Argus Search**: Central search API via HTTP (`http://100.112.130.100:8270/api/search`) or MCP (`mcp__argus__search_web`).
- **Status Note**: Janitor automated background session analysis is disabled pending redesign.

## Canonical entry points

- [`README.md`](README.md) — Primary human handbook and fast-path commands (`hl onboard`, `hl remote-status`)
- [`docs/README.md`](docs/README.md) — Documentation index and architectural navigation
- [`docs/OPERATIONS.md`](docs/OPERATIONS.md) — Operational runbooks and human fallback procedures
- [`docs/ROADMAP.md`](docs/ROADMAP.md) — Control-plane completion roadmap and verification gates
- [`docs/HOMELAB_API.md`](docs/HOMELAB_API.md) — REST API specification for Homelab health and observations
- [`docs/SECRETS.md`](docs/SECRETS.md) — SOPS/Age encryption policy and secret workflows
- [`docs/INFRA_STATE.md`](docs/INFRA_STATE.md) — Machine topology and infrastructure state
- [`docs/reference/BAYWATCH.md`](docs/reference/BAYWATCH.md) — Health watcher workflow, isolation policy, and observation specification
- [`docs/reference/SCHEDULED-JOBS.md`](docs/reference/SCHEDULED-JOBS.md) — Systemd backup and scanner timer schedules
- [`docs/reference/PROJECT_INDEX.md`](docs/reference/PROJECT_INDEX.md) — Ownership catalog and monitoring state mapping
- [`AGENTS.md`](AGENTS.md) — ONE_SHOT v14 agent orchestration contract, intelligence tiers, and routing table
- [`homelab.yaml`](homelab.yaml) — Project metadata specification (`homelab.project/v1`)
- [`docker-compose.yml`](docker-compose.yml) — Tiered Docker service definitions
- [`Makefile`](Makefile) — Management, deployment, and audit targets
- [`llms.txt`](llms.txt) — Concise machine-readable overview
- [`llms-full.txt`](llms-full.txt) — Extended machine context briefing
