# LLM-OVERVIEW — archon
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is

Archon is a container-oriented application repository combining a web UI, Python/backend code, agent and MCP-facing services, documentation, deployment automation, migrations, and a separately documented secret-vault capability. The repository is operationally driven by Docker Compose, `Makefile` targets, shell deployment scripts, environment variables, and service-specific compositions.

`AGENTS.md` is the governing agent contract. It describes ONE_SHOT v14 orchestration, worker selection, dispatch, search, and operator workflows; those rules govern work on this repository but are not evidence that the corresponding orchestration components are part of the Archon runtime. In particular, worker commands, Argus search, and `/short`, `/full`, or `/conduct` are agent operating procedures rather than Archon application entry points.

Current runtime health is unknown: the live status probe was disabled. Git history shows continued repository maintenance through the Homelab project contract and ONE_SHOT framework synchronization, but does not independently prove that any deployed instance is presently reachable.

## Machine & Host Ownership

No verified Archon runtime machine or multi-host deployment ownership is available from the disabled status probe or the supplied `AGENTS.md`; its `oci-ts`, `macmini-ts`, and Argus homelab references describe orchestration/search infrastructure, not confirmed Archon hosts.

## What is actually built

Evidence from tracked top-level paths and commit history supports these implemented areas:

- **Containerized application stack:** root `docker-compose.yml`, additional `compositions/`, port-oriented environment configuration, and Docker-focused operations.
- **Web frontend:** `archon-ui-main/`, with a public application graphic under its `public/` tree and Vite-related configuration implied by `VITE_ALLOWED_HOSTS` and `VITE_SHOW_DEVTOOLS`.
- **Backend/runtime code:** `python/`, `modules/`, `bin/`, `serve-solution.py`, and `test_backend.py`.
- **Distinct service surfaces:** configured ports for agents, documentation, MCP, server/API, and UI: `ARCHON_AGENTS_PORT`, `ARCHON_DOCS_PORT`, `ARCHON_MCP_PORT`, `ARCHON_SERVER_PORT`, and `ARCHON_UI_PORT`.
- **Supabase integration:** runtime configuration requires or recognizes `SUPABASE_URL` and `SUPABASE_SERVICE_KEY`.
- **Observability and runtime controls:** `LOGFIRE_TOKEN`, `LOG_LEVEL`, `PROD`, and `HOST`.
- **Database/schema evolution:** `migration/`.
- **Deployment and operations:** `deploy-archon.sh`, `LOGIN.sh`, `Makefile`, `check-env.js`, `scripts/`, and `WORKING_DEPLOYMENT_CONFIG.md`.
- **Reverse-proxy/public exposure documentation:** `CADDY_INSTRUCTIONS_PUBLIC.md` and `UNIVERSAL_CADDY_SOLUTION.md`.
- **Secret vault capability:** `AI_VAULT_INSTRUCTIONS.md`, `VAULT_SETUP.sh`, `VAULT_DEPLOYMENT_COMPLETE.md`, and commits documenting a vault web interface, security configuration, production deployment, and secret-update procedures.
- **Project integrations and design rationale:** `OOS_INTEGRATION_REFERENCE.md` and commits documenting an OOS dependency system and principles.
- **Planning/reference material:** `PRPs/`, `DEVELOPER_REFERENCE.md`, `QUICK_START.md`, `DOMAIN_PORT_REGISTRY.md`, and `FORK_MAINTENANCE.md`.
- **Project contract metadata:** `homelab.yaml`, added or formalized by the latest supplied commit.
- **Contributor documentation:** `README.md`, `CONTRIBUTING.md`, `CLAUDE.md`, and `AGENTS.md`.

The commit history supports that the vault and OOS integration were implemented historically. Because no status probe ran, treat deployment-completion documents and commit subjects as repository history, not current service-health evidence.

## Canonical entry points

Read or invoke these in this order according to the task:

1. **Agent rules:** `AGENTS.md`
   - Defines vocabulary, task classes, routing lanes, worker preference, dispatch protocol, and operator modes.
   - Its orchestration instructions take precedence over this derived overview.
2. **Repository-specific assistant context:** `CLAUDE.md`
   - Consult before implementation for Archon-specific development constraints not reproduced here.
3. **User-facing project orientation:** `README.md`
   - Primary description and navigation document.
4. **Fast setup:** `QUICK_START.md`
   - Expected starting point for installation or initial launch.
5. **Comprehensive development reference:** `DEVELOPER_REFERENCE.md`
   - Preferred source for detailed development procedures and architectural rationale.
6. **Service orchestration:** `docker-compose.yml`
   - Root container topology and service wiring.
7. **Operational commands:** `Makefile`
   - Preferred command façade when an applicable target exists.
8. **Deployment:** `deploy-archon.sh`
   - Deployment automation; inspect configuration and target assumptions before execution.
9. **Environment validation:** `check-env.js`
   - Validates or reports required runtime configuration.
10. **Backend/application runtime:** `python/`, `modules/`, `bin/`, `serve-solution.py`
    - Inspect actual code and imports before selecting a direct executable.
11. **Frontend:** `archon-ui-main/`
    - UI source, build configuration, and public assets.
12. **Alternate compositions:** `compositions/`
    - Deployment variants or service groupings; do not assume they are interchangeable with root Compose.
13. **Schema/data changes:** `migration/`
    - Review before modifying persistence models or Supabase-backed structures.
14. **Backend smoke/test entry:** `test_backend.py`
    - Existing backend test surface; determine its test runner and external-service requirements from the file.
15. **Vault setup:** `VAULT_SETUP.sh` and `AI_VAULT_INSTRUCTIONS.md`
    - Dedicated setup and operating instructions for the secret-vault subsystem.
16. **Public proxy setup:** `CADDY_INSTRUCTIONS_PUBLIC.md` and `UNIVERSAL_CADDY_SOLUTION.md`
    - Reverse-proxy and public-routing guidance.
17. **Network allocation:** `DOMAIN_PORT_REGISTRY.md`
    - Canonical registry for domains and ports; update it when service exposure changes.
18. **Homelab contract:** `homelab.yaml`
    - Machine/deployment metadata source; validate its schema and current contents before relying on it operationally.

## Architectural map

### Application services

The declared environment surface indicates at least five separately addressable roles:

| Role | Configuration | Evidence-backed responsibility |
|---|---|---|
| Agent service | `ARCHON_AGENTS_PORT` | Agent-facing service endpoint. Exact protocol and implementation require source inspection. |
| Documentation service | `ARCHON_DOCS_PORT` | Hosted documentation endpoint. |
| MCP service | `ARCHON_MCP_PORT` | Model Context Protocol endpoint or gateway. |
| Server service | `ARCHON_SERVER_PORT` | Main backend/API endpoint. |
| UI service | `ARCHON_UI_PORT` | Browser-facing frontend endpoint. |

Do not collapse these into one process without inspecting `docker-compose.yml`: separate port variables imply distinct network surfaces but do not prove distinct containers or executables.

`HOST` controls bind or advertised host behavior. `PROD` selects a production-oriented runtime path. `LOG_LEVEL` controls logging verbosity. `LOGFIRE_TOKEN` implies optional or required Logfire telemetry. Confirm defaults, validation, and production requirements in Compose, `check-env.js`, and backend configuration.

### Frontend

`archon-ui-main/` is the canonical frontend tree. The `VITE_*` variables identify a Vite-based browser build:

- `VITE_ALLOWED_HOSTS` controls development/server host acceptance or proxy exposure.
- `VITE_SHOW_DEVTOOLS` controls a browser-visible developer-tool surface.
- Public assets include `archon-main-graphic.png`.

When changing frontend-to-backend communication, verify all three layers together:

1. frontend environment/configuration;
2. Compose service names and exposed ports;
3. Caddy/public-domain routing.

Do not infer that browser code may safely consume server secrets. `SUPABASE_SERVICE_KEY` is privileged server configuration and must not be exposed through Vite-prefixed variables or frontend bundles.

### Backend and modules

Backend implementation is distributed across `python/`, `modules/`, `bin/`, and root Python entry files. Establish ownership from imports and Compose commands before editing:

- `python/` likely contains the main Python packages or service implementations.
- `modules/` contains modular capabilities or extensions.
- `bin/` contains executable wrappers or operational utilities.
- `serve-solution.py` is a direct serving entry candidate.
- `test_backend.py` is the visible root backend verification surface.

These path names alone do not establish package boundaries, frameworks, process models, or stable APIs. Use actual imports, container commands, and module metadata as authority.

### Persistence and Supabase

The presence of `SUPABASE_URL`, `SUPABASE_SERVICE_KEY`, and `migration/` establishes Supabase-backed persistence or service integration.

Operational invariants:

- Treat the service key as a secret.
- Keep schema-changing code and migrations synchronized.
- Check whether migrations are applied by Compose startup, deployment scripts, a manual command, or Supabase tooling before altering them.
- Verify authorization behavior at the externally observable API boundary; possession of the service key commonly bypasses client-level access controls.
- Do not assume a local database exists merely because migrations are tracked.

### Secret vault

The vault subsystem has dedicated setup, access, public-routing, deployment-completion, and secret-update documentation. Commit history records:

- secret-vault implementation;
- web interface and security configuration;
- production deployment confirmation;
- simplified secret-update instructions;
- AI access documentation.

Canonical vault materials:

- `AI_VAULT_INSTRUCTIONS.md` — agent/user access workflow.
- `VAULT_SETUP.sh` — setup automation.
- `VAULT_DEPLOYMENT_COMPLETE.md` — historical deployment record.
- `CADDY_INSTRUCTIONS_PUBLIC.md` — public proxy instructions.
- `WORKING_DEPLOYMENT_CONFIG.md` — known working configuration.

Security boundary: never copy credentials, service keys, vault contents, or authenticated URLs into documentation, logs, prompts, commits, frontend environment variables, or generated overviews. Deployment-completion claims are historical until a live probe verifies the service.

### Compose and deployment

`docker-compose.yml` is the root topology; `compositions/` contains additional topology variants. Before operating the stack:

1. inspect the selected Compose file;
2. identify required environment files and secrets;
3. run the repository’s environment validator;
4. consult `DOMAIN_PORT_REGISTRY.md` for conflicts;
5. check Caddy/public-domain expectations;
6. use existing `Makefile` or deployment-script commands rather than constructing a parallel procedure.

`deploy-archon.sh` and `LOGIN.sh` are privileged operational surfaces. Inspect targets, remote hosts, credential sources, destructive actions, and idempotency before running them. Do not infer deployment ownership from their names.

### Reverse proxy and network exposure

Caddy-related documents indicate a supported public reverse-proxy path. Network changes must remain consistent across:

- container listen ports;
- host-published ports;
- `DOMAIN_PORT_REGISTRY.md`;
- Caddy routes;
- allowed-host configuration;
- frontend API/MCP endpoint configuration;
- firewall or homelab configuration where documented.

A service being healthy inside Compose does not prove public reachability. Conversely, a Caddy route existing does not prove the backing container is healthy.

### OOS integration

`OOS_INTEGRATION_REFERENCE.md` and associated commits document an integration dependency system plus architectural principles. Treat that document as the integration contract. Determine whether OOS is a runtime dependency, development dependency, or documentation/process integration from its current contents before modifying coupling.

Do not conflate OOS integration with the ONE_SHOT orchestration rules in `AGENTS.md`; they may be related historically, but they occupy different evidence surfaces.

### Plans and project records

`PRPs/` contains project requirement or planning records. Plans describe intent and constraints; they are not evidence that a feature is currently implemented. Cross-check planned behavior against source, Compose, migrations, and recent commits.

`VAULT_DEPLOYMENT_COMPLETE.md` and `WORKING_DEPLOYMENT_CONFIG.md` are operational records. Preserve useful provenance, but do not treat names such as “complete,” “final,” or “working” as live-health assertions.

## Development and operations workflow

### Before changes

- Read `AGENTS.md`, then `CLAUDE.md`.
- Locate the relevant service in `docker-compose.yml`.
- Inspect the implementation entry point and all callers.
- Consult `DEVELOPER_REFERENCE.md` and subsystem-specific documentation.
- For port/domain changes, consult `DOMAIN_PORT_REGISTRY.md`.
- For persistence changes, inspect `migration/` and Supabase configuration.
- For public exposure, inspect both Caddy documents.
- For fork/upstream work, follow `FORK_MAINTENANCE.md`.

### Common command surfaces

- `make …` — preferred repository-defined operations; enumerate targets from the Makefile.
- `docker compose …` — root service lifecycle; use the exact profiles/files documented by the repository.
- `node check-env.js` — likely environment validation; confirm invocation in its source or Makefile.
- `python3 serve-solution.py` — possible direct service launch; confirm arguments and dependencies first.
- `python3 test_backend.py` or the configured Python test runner — backend verification; inspect test conventions before execution.
- `./deploy-archon.sh` — deployment path; inspect before use.
- `./VAULT_SETUP.sh` — vault setup; inspect environment and privilege requirements before use.

Commands above are entry candidates derived from filenames, not guaranteed argument contracts. Repository scripts and Make targets are authoritative.

### Verification expectations

For backend changes, exercise the affected API or service, not only imports. For frontend changes, run the UI and verify the changed browser path against the configured backend. For Compose changes, validate container startup, health, internal service resolution, and the externally exposed route. For migrations, verify application behavior against the migrated schema. For vault or authentication changes, verify denial as well as successful access without printing secrets.

## Dependencies and configuration

### Confirmed configuration names

- `ARCHON_AGENTS_PORT`
- `ARCHON_DOCS_PORT`
- `ARCHON_MCP_PORT`
- `ARCHON_SERVER_PORT`
- `ARCHON_UI_PORT`
- `HOST`
- `LOGFIRE_TOKEN`
- `LOG_LEVEL`
- `PROD`
- `SUPABASE_SERVICE_KEY`
- `SUPABASE_URL`
- `VITE_ALLOWED_HOSTS`
- `VITE_SHOW_DEVTOOLS`

The supplied evidence does not establish which variables are mandatory, their defaults, valid formats, or whether they belong in one shared environment file. Resolve those facts from `docker-compose.yml`, `check-env.js`, frontend configuration, and backend settings.

### External systems supported by evidence

- Docker/Compose runtime.
- Supabase.
- Logfire.
- Caddy or a Caddy-compatible public reverse-proxy deployment.
- OOS integration.
- Browser/Vite frontend.
- MCP-facing service surface.

`AGENTS.md` additionally references Argus, OpenRouter, Codex, Gemini CLI, GLM-backed Claude tooling, and SSH workers. Those are development/orchestration facilities unless Archon source or deployment configuration explicitly imports them.

## Repository state

Latest supplied commit: `b14c587 chore: add Homelab project contract`.

Recent maintenance is dominated by:

- Homelab contract addition.
- ONE_SHOT framework synchronization.
- LLM overview bootstrapping.
- Earlier vault, Caddy, OOS, and documentation work.

No live status result is available. No current blocker or issue list was supplied. Absence of a reported blocker is not proof that all services build or run.

## High-risk boundaries and gotchas

- **Authority:** `AGENTS.md` governs agent behavior; this file only compresses repository evidence.
- **Generic orchestration versus application runtime:** ONE_SHOT workers, Argus, and SSH dispatch examples are not automatically Archon dependencies.
- **Disabled health probe:** do not report the project or deployment as healthy, active, or down without a new runtime check.
- **Historical deployment language:** “complete,” “final,” “production,” and “working” appear in filenames or commit subjects; treat them as historical claims.
- **Secrets:** never expose `SUPABASE_SERVICE_KEY`, vault data, Logfire credentials, deployment credentials, or authenticated endpoints.
- **Frontend environment:** only intentionally public values may enter Vite-prefixed configuration.
- **Port coordination:** update Compose, domain registry, proxy configuration, allowed hosts, and clients together.
- **Multiple deployment surfaces:** root Compose, `compositions/`, Make targets, deployment scripts, and Caddy documents can drift; identify the selected deployment path before changing it.
- **Plans are not implementation:** `PRPs/` and design/reference documents require source confirmation.
- **Top-level Python ambiguity:** do not assume `serve-solution.py` or `test_backend.py` represents every backend service.
- **Supabase migrations:** schema, authorization policy, generated clients, and backend expectations may need coordinated updates.
- **Fork maintenance:** consult `FORK_MAINTENANCE.md` before rebasing, importing upstream work, or changing vendored/upstream-owned code.
- **Host ownership:** `homelab.yaml` may contain relevant deployment metadata, but no host assignment is verified in the supplied status evidence.
