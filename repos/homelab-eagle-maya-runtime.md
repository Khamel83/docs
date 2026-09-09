# LLM-OVERVIEW — homelab-eagle-maya-runtime
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is

homelab-eagle-maya-runtime (personal multi-machine `homelab` workspace) is the infrastructure source of truth, host automation, service catalog, and health monitoring environment for personal multi-machine operations. It owns Docker container deployments across 3 stability tiers, SOPS-encrypted secrets management, systemd maintenance/backup schedules, project catalog reconciliation (63 registered projects mapped against Hestia), and `homelab-api` / Baywatch control-plane diagnostic flows.

The repository is human-operable without an LLM via Makefile targets, standard health REST endpoints, and the `hl` front-door CLI (`scripts/homelab_cli.py`). It also hosts the ONE_SHOT v14 orchestration contract (`AGENTS.md`), governing multi-model worker dispatching (`dispatch-run`, `codex`, `gemini`, `v4-flash`, `mm2.7`/`mm2.5`) and local search execution via Argus.

## Machine & Host Ownership

- **Mac mini**: Primary development host and native-runtime host. Workspace editing, local verification, and worker dispatch origin.
- **Homelab**: Production Docker host, durable state host, backup target, and execution host for `homelab-api` and the Baywatch watcher.
- **MBA / work MBP**: Access client machines reaching Mac mini or Homelab via Tailscale and SSH; non-runtime hosts.
- **OCI**: Retained standby/legacy host (mapped to `deprecated` tier in placement audits); not active production truth.

## What is actually built

### 1. Front-Door CLI & Project Inventory (`scripts/homelab_cli.py`, `Makefile`, `hestia/`)
- **`hl` CLI Tool**: Front-door interface (`hl onboard`, `hl remote-status`, `hl help`) delegating execution to Makefile targets.
- **Project Catalog & Hestia Registry**: Reconciles 63 catalog entries (`config/project-catalog.yml`) with Hestia registrations (`hestia/registry.yml`). Catalog states (`managed`, `observing`, `standby`, `archived`) define Baywatch monitoring scope.
- **Audit Tooling**: Static read-only auditing via `make project-audit`, `make state-status`, and `make repo-state-status` (`tests/test_project_catalog_audit.py`, `tests/test_verify_registry.py`).

### 2. Multi-Tier Service Architecture (`docker-compose.yml`, `services/`, `scripts/regen-service-tiers.py`)
- **Tier 1 (Critical)**: Core network, reverse proxies, DNS, and authentication (`authentik`). Manual updates only.
- **Tier 2 (Stable)**: Auto-update safe applications, media services, and observability apps (`apprise`, `audiobookshelf`, `autoheal`).
- **Tier 3 (Experimental)**: Opt-in test containers.
- **Core Microservices (`services/`)**: `atlas-api`, `atlas-edge-consumer`, `atlas-edge-worker`, `atlas-postgres`, `atlas-tunnel`, `argus` (search API at `http://100.112.130.100:8270/api/search`), `baywatch`.
- **Tier Classifier**: `scripts/regen-service-tiers.py` maps machine hosts to tier environments (`homelab` -> `production`, `staging` -> `staging`, `deprecated` -> `deprecated`).

### 3. Control-Plane, Health & Baywatch Monitoring (`services/baywatch/`, `homelab.yaml`, `tests/test_baywatch_deployment.py`)
- **`homelab-api`**: Public `GET /health` endpoint separates informational `services_stopped` from actionable `services_unhealthy`. Authenticated routes handle observation ingest (`POST /observations`) and incident polling (`GET /incidents`). Wide mutation endpoints (`POST /services`, `DELETE /services/{name}`) are retired (return 410).
- **Baywatch Watcher**: Hardened 5-minute systemd oneshot timer (`baywatch.service`, `baywatch.timer`) running under unprivileged user `baywatch`. Configured with `NoNewPrivileges=true`, empty bounding capability set, no LLM execution (`--no-llm`), file locking (`/var/lib/baywatch/baywatch.lock`), and no Docker socket/SSH/shell access. Exit code `3` flags detected incidents and counts as diagnostic success (`SuccessExitStatus=3`).

### 4. Secrets Vault & SOPS Workflow (`secrets/`, `Makefile`)
- **SOPS Vault**: Encrypted `.env.encrypted` credentials stored in `secrets/` (`homelab.env.encrypted`, `maya.env.encrypted`, `atlas.env.encrypted`, `services.env.encrypted`, `openclaw.env.encrypted`).
- **Secret Helpers**: Encrypted/decrypted using SOPS and age keys via `make encrypt-secrets`, `make decrypt-secrets`, `make edit-secrets`, and `make test-sops`. Plaintext secrets in source control are strictly forbidden.

### 5. Automated Backups & Systemd Timers (`systemd-timers/`, `crontabs/`)
- **Backup Services**: Systemd timers for host backups (`backup-main.timer`, `backup-main2.timer`, `backup-maya.timer`).
- **Host Maintenance**: Periodic file/system scanning timers (`clamav-scan.timer`, `czkawka-scan.timer`).
- **Crontab Distribution**: Host-specific crontab files (`crontabs/crontab.homelab`, `crontabs/crontab.macmini`, `crontabs/crontab.mba`) deployed via `install-crontabs.sh`.

### 6. Agent Dispatch & Intelligence Infrastructure (`AGENTS.md`, `config/`)
- **Routing Protocol**: `AGENTS.md` (ONE_SHOT v14) routes tasks across model classes via `dispatch-run` (`~/.local/bin/dispatch-run`).
- **Worker Tiers**: `v4-flash` (DeepSeek v4-flash), `codex` (ChatGPT Plus CLI), `gemini` (Google AI Pro CLI), `mm2.7`/`mm2.5` (MiniMax models via `patches/oc-go-cc`).
- **Search Integrations**: Argus search API (`http://100.112.130.100:8270/api/search`) and MCP tool (`mcp__argus__search_web`).
- **Janitor Status**: Disabled pending redesign.

### 7. Dashboard & UI Configuration (`homepage/`)
- Homepage dashboard configuration managing Docker integration, Proxmox views, bookmark catalog, and service widgets (`homepage/services.yaml`, `homepage/custom.js` 30-second status polling loop).

## Canonical entry points

- `AGENTS.md` — Agent constitution, task classification routing table, and CLI shortcuts
- `README.md` — Human operational entry point and fast-path commands (`hl onboard`, `hl remote-status`)
- `Makefile` — Build, deployment, vault decryption, catalog auditing, and diagnostic targets
- `docker-compose.yml` — Three-tier Docker service deployment specification
- `homelab.yaml` — Root project declaration (`homelab.project/v1`)
- `pyproject.toml` — Python script environment definitions and pytest configuration
- `docs/README.md` — Active documentation handbook index
- `docs/OPERATIONS.md` — Manual operational procedures and health verification fallback
- `docs/ROADMAP.md` — Active development milestones and acceptance criteria
- `docs/HOMELAB_API.md` — Health, observation, and incident API specifications
- `docs/SECRETS.md` — SOPS/age secret encryption and key handling procedures
- `llms.txt` / `llms-full.txt` — Machine-readable entry points for external agent intake
