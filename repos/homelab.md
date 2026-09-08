# LLM-OVERVIEW — homelab
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is

Homelab is the single infrastructure source of truth for a personal multi-machine environment. It defines and governs production Docker deployments, durable system state, backup policies, health evidence ingestion, scheduled operations, secrets workflows, and a normalized catalog of 63 registered projects across `homelab.yaml` and Hestia integrations.

The repository houses container Compose manifests, Ansible playbooks, systemd timers, cron configurations, dotfiles, scripts, and the `hl` control-plane CLI. Production workloads and health monitoring are orchestrated around the Homelab API and the Baywatch systemd observer, alongside specialized infrastructure modules including Maya, Argus, Hermes, and Free99.

## Machine & Host Ownership

Machine roles and runtime boundaries are declared as follows in `AGENTS.md`:

- **Mac mini:** Primary development host and owner of native macOS runtimes. Local code development and static audits occur here.
- **Homelab:** Primary production Docker host. Owns durable state, backup operations, the Homelab API service, and scheduled infrastructure diagnostics.
- **MBA and work MBP:** Client access machines by default (via Tailscale and SSH); not default runtime hosts.
- **OCI (Oracle Cloud Infrastructure):** Retained standby and legacy capacity only. New workloads are not placed here, and historical OCI records do not represent current live placement.

Private operator access uses Tailscale and SSH. Public application ingress is managed via Cloudflare routing as documented in `docs/ARCHITECTURE.md`. Tailscale Funnel and legacy routing stacks are retired and not deployment defaults.

## What is actually built

The repository integrates several distinct control-plane subsystems, orchestration stacks, and operational tools:

### 1. Control Plane & Baywatch Health Monitoring
- **Baywatch Observer:** A pinned, read-only systemd watcher running every 5 minutes on Homelab. It queries local project definitions (`/projects`, `/projects/{id}`) and posts sanitized observations to `/observations`.
- **Exit Codes:** Exit `0` indicates healthy execution with no incidents; exit `3` indicates incidents were found (treated as a successful systemd diagnostic run); exit `1` indicates run or observation delivery failure; exit `2` indicates invalid configuration.
- **Homelab API & Persistence:** Stores observations and a durable outbox in SQLite. Exposes authenticated project views, incident polling (`/incidents?cursor=...`), and observation ingest.
- **Health Surface:** `GET /health` is public. It distinguishes informational `services_stopped` from active `services_unhealthy` (exited containers alone do not mark Homelab degraded).
- **Retired Endpoints:** Broad `POST /services` and `DELETE /services/{name}` are retired (return HTTP 410 Gone).
- **Security Boundaries:** Baywatch has no Docker socket, SSH key, age identity, shell capability, or direct Maya connection.

### 2. Maya Subsystem (Canonical Core Migration Target)
- **Profile-Gated Core Compose:** Profile-gated canonical core Compose stack (`api`, `scheduler`, `postgres`) targeting Homelab migration (`bb08507`).
- **Standalone Network:** Configured with its own standalone network (no root Compose include until cutover).
- **API Endpoint & Environment:** Published on `0.0.0.0:18200` for client repoint (rehearsal port 18200; host process owns port 8200). Passes full root `.env` environment through to role containers.

### 3. Argus Subsystem (Vault Projections & Ledger Verification)
- **Provider Vault Projections:** Supports persistent provider vault projections, atomic projection repairs, and explicit provider runtime registrations while skipping absent legacy vault fallbacks.
- **Promotion & Scorecard:** Features scorecard network subnet preflight checks, accepted-operation ledger verification on promotion, preservation of release and recovery identities across promotion, and verified projection restorations.

### 4. Hermes Subsystem (Task-Ingress Bridge)
- **Task-Ingress Bridge:** Exposes task-ingress routing and dedicated HTTP read routes for task status consumers.

### 5. Free99 Subsystem (NVIDIA Endpoint Catalog)
- **Catalog Integration:** Contains the NVIDIA free endpoint catalog and endpoint deployment configurations.

### 6. Project Catalog & Service Management
- **Catalog Scope:** 63 registered projects categorized into `managed`, `observing`, `standby`, and `archived`.
- **Monitoring Scope:** `managed` and `observing` entries represent active Baywatch monitoring scope. `standby` and `archived` entries remain visible in records but are quiet.
- **Audit Tooling:** `make project-audit` statically reconciles project definitions, Hestia configurations, and service ownership without claiming live health.

### 7. Infrastructure, Automation & Repository CLI
- **`hl` CLI Tooling:** Core repository CLI supporting local and remote operations (`hl onboard`, `hl doctor-dev-tools-local`, `hl remote-status`, `hl cron-status`, `hl remote-recreate-SERVICE`).
- **Ansible & Config Files:** Playbooks and roles (`ansible/`, `infra/`, `config/`) for system provisioning, dotfiles management (`dotfiles/`), and systemd timer installation (`systemd-timers/`, `cron.d/`, `crontabs/`).
- **Secrets Architecture:** Uses SOPS with age keys as detailed in `docs/SECRETS.md`. Plaintext secrets, raw age keys, and API tokens are excluded from commit history.

## Canonical entry points

- `AGENTS.md` — Single shared agent operating contract, authority order, and vocabulary rules.
- `docs/guides/INFRA_ACCESS.md` — Approved runtime access onboarding and credential guidelines.
- `docs/README.md` — Active handbook and documentation index.
- `docs/ARCHITECTURE.md` — Network topology, Cloudflare routing, and service boundaries.
- `docs/OPERATIONS.md` — Human operational runbooks, recovery steps, and bounded health probes.
- `docs/ROADMAP.md` — Remaining implementation work, milestones, and acceptance gates.
- `docs/HOMELAB_API.md` — Homelab API endpoint specifications and data models.
- `docs/reference/BAYWATCH.md` — Baywatch architecture, outbox processing, and exit status definitions.
- `docs/SECRETS.md` — SOPS encryption workflow and secrets management model.
- `docs/reference/SCHEDULED-JOBS.md` — Inventory of systemd timers, cron entries, and background jobs.
- `docs/reference/PROJECT_INDEX.md` — Normalized project catalog index and service ownership mapping.
