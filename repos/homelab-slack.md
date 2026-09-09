# LLM-OVERVIEW — homelab-slack
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
Homelab is the declarative infrastructure source of truth and management hub for a personal multi-machine environment. It owns production Docker service orchestrations, durable state storage, system backups, scheduled diagnostics, security secrets, and the normalized catalog of 63 registered projects across `managed`, `observing`, `standby`, and `archived` lifecycles. It provides a 3-tier container deployment architecture (Critical, Stable, Experimental), declarative project metadata contracts (`homelab.yaml`), project onboarding CLI tools (`hl`), secret management via SOPS/age, Hestia registry synchronization, Cloudflare/Tailscale Terraform configurations, Ansible provisioning playbooks, and systemd maintenance timers. Active control-plane components include the Baywatch health observer and the native Hermes projection loop for Maya receipt and drop graph delivery.

## Machine & Host Ownership
- **Mac mini**: Primary development host and owner of native macOS runtimes. Dedicated environment for code editing, script execution, local test suites, and CLI execution (`hl`).
- **Homelab**: Production Docker host, durable state repository, backup host, and owner of the Homelab API (`homelab.api`). Host for Baywatch systemd observer and production services.
- **MBA / Work MBP**: Operator access clients connecting via SSH and Tailscale. Not required runtime hosts.
- **OCI (Oracle Cloud Infrastructure)**: Retained standby/legacy host capacity. Classified as `deprecated` host tier in placement auditing (`audit-placement.sh`, `regen-service-tiers.py`). No new workloads are placed here.

## What is actually built
- **Tiered Container Architecture (`docker-compose.yml`, `services/`, `homepage/`)**:
  - Shared bridge network `homelab` (`172.20.0.0/16`).
  - **Tier 1 (Critical)**: DNS, proxy, and core management containers. Updated manually only.
  - **Tier 2 (Stable)**: Auto-update safe applications and background services located in `services/` (Apprise, Argus, Atlas API/Edge Consumer/Edge Worker/Postgres/Tunnel, Audiobookshelf, Authentik, Autoheal).
  - **Tier 3 (Experimental)**: Opt-in testing services.
  - **Homepage Dashboard (`homepage/`)**: Custom JS (`custom.js`) auto-refresh (30s interval) monitoring 40+ containerized services across homelab and cloud infrastructure.
- **Project Catalog & CLI Onboarding Tooling (`scripts/homelab_cli.py`, `homelab.yaml`, `pyproject.toml`, `hestia/`)**:
  - `hl` CLI (`scripts/homelab_cli.py`): Onboards projects (`hl onboard`), checks status (`hl remote-stat`), and executes diagnostic suites (`hl doctor-dev-tools-local`).
  - Onboarding pipeline (`test_homelab_cli.py`): Validates `docker-compose.yml`, publishes declarative `homelab.yaml` metadata (schema `homelab.project/v1`) and Hestia registry entries (`hestia/registry.yml`), and triggers initial observation ingest.
  - Python stack (`pyproject.toml`): Requires Python >=3.10, `jsonschema==4.26.0`, `pydantic==2.13.4`, with `pytest`, `PyYAML`, and `ruamel.yaml` for dev/test execution.
- **Hermes Projection Loop & Maya Integration (`scripts/`, `tests/test_compile_drop_graph.py`)**:
  - Native Hermes projection loop engine projecting exact receipt envelopes and atomic drop graphs to Maya.
  - Verification & Safety: Validates Maya receipt envelopes before delivery, verifies Maya acknowledgement request hashes, dead-letters exhausted delivery retries, and includes a projector recovery safety harness.
- **Argus Image Promotion & Contract State (`scripts/argus-promotion-state.py`, `tests/test_argus_promotion_contract.py`)**:
  - Immutable Digest Enforcement: Requires digest-addressed image refs (`ghcr.io/khamel83/argus@sha256:...`) and strictly rejects mutable tags (e.g. `:latest`).
  - Contract validation suites cover promotion state, live scorecard contracts, and Postgres recovery contracts.
- **Placement Auditor & Tier Maintenance (`scripts/audit-placement.sh`, `scripts/regen-service-tiers.py`, `tests/test_placement_auditor.py`)**:
  - Compares running Docker containers against `config/service-tiers.yml` and `config/llm-overview-tiers.json`. Evaluates drift across machine tiers (`homelab` -> `production`, `staging` -> `staging`, `deprecated` -> `deprecated`).
- **Infrastructure as Code & Automation (`infra/`, `ansible/`, `secrets/`, `systemd-timers/`)**:
  - **Terraform (`infra/terraform`)**: Declarative state for Cloudflare ingress/routing and Tailscale networking.
  - **Ansible (`ansible/`)**: Machine provisioning playbooks (`mac-mini-playbook.yml`, `dev-machines-playbook.yml`, `playbook.yml`).
  - **Secrets Broker (`secrets/`)**: SOPS/age encrypted `.env.encrypted` credentials (`arb`, `atlas`, `clio`, `convex`, `float`, `free99`, `github-runner-fleet`, `homelab-operator`).
  - **Systemd & Crons (`systemd-timers/`, `crontabs/`)**: Systemd timers for main backups (`backup-main`, `backup-main2`, `backup-maya`), ClamAV scans (`clamav-scan`), and Czkawka deduplication scans (`czkawka-scan`).
- **Control-Plane, Baywatch & Homelab API**:
  - **Baywatch**: Read-only systemd watcher on Homelab (5-minute timer). Reads `/projects` and `/projects/{id}`, outputs sanitized events to `/observations` with SQLite outbox persistence.
  - **Homelab API Ingress**: Public `GET /health` separating informational `services_stopped` from `services_unhealthy`. Authenticated endpoints for catalog metadata, observations, and incident polling (`/incidents?cursor=...`). Broad service mutation endpoints (`POST /services`, `DELETE /services/{name}`) are retired (410 Gone).

## Canonical entry points
- [`AGENTS.md`](AGENTS.md) — Single shared agent operating contract and vocabulary
- [`docs/README.md`](docs/README.md) — Active documentation index and handbook
- [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) — Ingress, networking, and system architecture
- [`docs/OPERATIONS.md`](docs/OPERATIONS.md) — Health status, live diagnostics, and human recovery procedures
- [`docs/HANDOFF.md`](docs/HANDOFF.md) — Durable record of open issues, active tasks, and session state
- [`docs/ROADMAP.md`](docs/ROADMAP.md) — Remaining work, execution sequence, and acceptance criteria
- [`docs/HOMELAB_API.md`](docs/HOMELAB_API.md) — API contract, event sinks, and observation formats
- [`docs/SECRETS.md`](docs/SECRETS.md) — SOPS/age secret model and machine-local broker workflow
- [`docs/INFRA_STATE.md`](docs/INFRA_STATE.md) — Infrastructure state inventory and classification status
- [`docs/guides/INFRA_ACCESS.md`](docs/guides/INFRA_ACCESS.md) — Approved runtime and network access onboarding
