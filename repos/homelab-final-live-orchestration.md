# LLM-OVERVIEW — homelab-final-live-orchestration
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is

Infrastructure and orchestration source of truth for the personal Homelab environment. The repository combines:

- production Docker service definitions and tiered lifecycle operations;
- machine configuration through Ansible, Terraform, cron, and systemd timers;
- the Homelab health/control API contract and bounded operational tooling;
- the Hestia project registry, project metadata, ownership, and monitoring declarations;
- Baywatch read-only project observation and incident production;
- Maya orchestration deployment configuration, webhook handoff, secret ownership, and backup contracts;
- encrypted secret material and its SOPS/age operating procedures;
- human-facing Homepage configuration;
- fleet onboarding, placement auditing, state inventory, backup, and deployment scripts.

`homelab.yaml` identifies this project as `homelab.project/v1`, project ID `homelab`, kind `service`, lifecycle `development`, owner `homelab`, monitoring state `standby`, and production unit `homelab` on host `homelab`. Its declared event sink is `homelab-api:/observations`; evidence must be sanitized. `managed_by: unknown`, an empty repair registry, and null `repair_id` are explicit declarations, not implementation gaps to infer around.

The repository is deliberately human-operable. Fixed commands, health endpoints, incident views, and deployment procedures remain authoritative without an LLM. Under the ONE_SHOT v14 contract in `AGENTS.md`, architecture and planning stay inline while implementation, research, review, drafting, and summarization are routed by task class to configured workers through `dispatch-run`. Argus is the designated web-search service. Janitor is disabled pending redesign and must not be treated as an active subsystem.

The catalog was last documented as 63 projects with matching Hestia registrations and no catalog parity errors. That establishes declared registration and ownership only; it is not live service-health evidence. The live status probe was disabled for this briefing, so current process, container, API, timer, webhook, and deployment health are unverified.

## Machine & Host Ownership

- **Homelab:** declared production host for Docker workloads, durable state, backups, infrastructure operations, and the Homelab API. `homelab.yaml` assigns the `homelab` production unit here. The root Compose network uses bridge subnet `172.20.0.0/16`.
- **Mac mini:** primary development and native-runtime host; Ansible and machine-specific crontab configuration are present.
- **MBA and work MBP:** access/development clients that reach Mac mini or Homelab over SSH/Tailscale; they are not required production runtime hosts.
- **OCI:** retained standby/legacy placement only. Old OCI records and tests mapping deprecated workloads to OCI must not be interpreted as current production placement.
- **Remote execution:** `AGENTS.md` documents worker dispatch over SSH to named machines, but these command examples do not prove that a worker or service is currently running there.
- **Maya:** repository configuration defines Maya orchestration production, Baywatch-to-Maya handoff, Maya-owned secrets, backup paths, deployment tests, and a corrected webhook route. Recent commits establish a finalized deployment contract; because no status probe ran, current runtime installation and health are not asserted.

## What is actually built

### Orchestration operating contract

`AGENTS.md` defines three operator flows:

- `/short`: load recent commits and project decision/blocker context, ask for the active task, then dispatch execution.
- `/full`: maintain `IMPLEMENTATION_CONTEXT.md`, perform structured intake and phase planning, dispatch parallel category-ordered work, and checkpoint context usage.
- `/conduct`: block on clarification, classify tasks, route each class to the first available preferred worker, dispatch non-premium work in parallel, and iterate until the stated goal is met.

The task router is `dispatch-run`, externally installed at `~/.local/bin/dispatch-run` from the Homelab scripts tree. It classifies through `v4-flash`, executes workers with `ProcessPoolExecutor`, and can synthesize parallel results. Defined classes are `plan`, `implement_small`, `implement_medium`, `implement_large`, `research`, `review_diff`, `doc_draft`, and `summarize_findings`; worker choices include Codex, Gemini, `v4-flash`, MiniMax M2.7, and MiniMax M2.5. This is an execution policy, not application runtime code in the Python package.

### Docker service plane

`docker-compose.yml` is the root service composition file. It creates the shared `homelab` bridge network and includes service-specific Compose definitions from `services/`. The update model has three operational tiers:

1. **Critical:** manual updates only for infrastructure whose failure can make the environment inaccessible.
2. **Stable:** normal auto-update-safe applications, media, and monitoring.
3. **Experimental:** opt-in services and new deployments.

The root file is an include-based composition layer rather than a monolithic list of container bodies. Service directories observed include Apprise, Argus, Atlas API/edge consumer/edge worker/Postgres/tunnel, Audiobookshelf, Authentik, and Autoheal; the repository tree may contain additional service directories beyond the abbreviated listing. Service ownership and tier behavior must be derived from the included Compose files and generated configuration, not from the Homepage dashboard list.

`Makefile` is the human command front door for starting, stopping, inspecting, pulling, and updating services. Important tier-aware targets include `up-critical`, `up-stable`, `up-all`, `update-critical`, and `update-stable`. Critical updates require deliberate handling; do not collapse all tiers into a single unattended update path.

### Homelab API and bounded control plane

The documented Homelab API exposes:

- public `GET /health`;
- authenticated project collection and project-detail views;
- observation ingestion at `/observations`;
- cursor-based incident polling and acknowledgement;
- bounded operational read views.

The health model separates informational stopped services from genuinely unhealthy services. Retained exited containers alone do not degrade Homelab. Broad `POST /services` and `DELETE /services/{name}` mutation endpoints are retired and return `410`; do not restore or route automation through them without changing the explicit API contract.

Homelab stores sanitized observations and a durable incident outbox in SQLite according to the documented control-plane contract. The observed repository extract does not expose the schema definitions or table names, so no additional database tables should be inferred here. The intended flow is project catalog → Baywatch observation → Homelab `/observations` → durable incident stream → human or scoped consumer. Homelab remains the bounded executor; arbitrary shell text and broad service mutation are outside the contract.

### Baywatch observation plane

Baywatch is represented by deployment configuration, a production example, systemd service/timer files, and `tests/test_baywatch_deployment.py`. The deployment contract enforces:

- a hardened `Type=oneshot` systemd service;
- a five-minute `OnUnitActiveSec=5min` timer;
- `User=baywatch`;
- `NoNewPrivileges=true`;
- an empty capability bounding set;
- a lock at `/var/lib/baywatch/baywatch.lock`;
- a 120-second start timeout;
- no dependency on `homelab-api.service`;
- no shell-enablement flag;
- `--no-llm`;
- no mutation authority.

Baywatch reads project catalog/detail data and submits sanitized observations to Homelab. It has no Docker socket, SSH key, age identity, arbitrary shell capability, or direct mutation path. Exit `3` means incidents were found and is configured as successful systemd diagnostic completion; exit `1` means execution or observation delivery failed; exit `2` means invalid configuration.

`config/baywatch-production.example.yml` is an example deployment input. `config/baywatch-maya-orchestration-handoff.yml` describes the Baywatch/Maya integration boundary. Configuration presence and deployment tests establish the contract, not current timer activity.

### Maya orchestration deployment

`config/maya-orchestration-production.yml` is the production orchestration definition. `config/baywatch-maya-orchestration-handoff.yml` defines its handoff from Baywatch/Homelab incidents. `tests/test_maya_orchestration_deployment.py` validates the deployment contract. Recent commits record:

- preparation of live Maya orchestration deployment;
- correction of Maya deployment contracts;
- separation of Maya and Hermes secret ownership;
- correction of the actual Maya webhook route;
- executable orchestration bootstrap;
- alignment of the final orchestration deployment contract.

This supersedes the older overview’s blanket statement that Maya incident consumption was merely planned. The repository now contains a deployment contract and integration configuration. No status probe ran, so active webhook delivery, consumer liveness, or successful end-to-end incident processing remains unverified.

`config/maya-backup-paths.txt`, `backup-maya.service`, and `backup-maya.timer` define Maya backup scope and scheduling. `secrets/maya.env.encrypted` is Maya’s encrypted secret bundle. Hermes and Maya secret ownership are separate; do not merge their credentials or assume one service owns the other’s material.

### Project catalog and Hestia

`homelab.yaml` is this repository’s normalized project record. The wider project inventory lives under `hestia/`, including:

- `registry.yml` for registrations;
- per-project declarations under `hestia/projects/`;
- templates and scripts for generation/auditing;
- reports and documentation;
- Hestia-specific tests.

Monitoring states distinguish active observation scope from quiet inventory. Historically documented `managed` and `observing` projects are active Baywatch scope; `standby` and `archived` projects remain registered but quiet. This repository itself declares `standby`.

`make project-audit` reconciles catalog declarations, Hestia registrations, and service ownership statically. It must not report or imply runtime health. `make project-graph` exposes dependency/ownership relationships. Onboarding is exposed through `hl onboard`; recent commits preserve existing registrations while adding the `hl` front door and local inventory registration.

### CLI and scripts

The Python package is metadata-only: `homelab-scripts` version `0.1.0`, Python `>=3.10`, with no runtime dependencies declared in `pyproject.toml`. Development dependencies are `pytest`, `PyYAML`, and `ruamel.yaml`; pytest discovers `tests/`.

`scripts/homelab_cli.py` implements the `hl` command surface. The observed CLI test proves legacy commands such as `remote-status` delegate to `make -C <homelab-root> remote-status`. Onboarding behavior is covered separately by `tests/test_homelab_onboard.py`.

Operational script groups include:

- service addition, deployment, health verification, notification, and environment generation;
- project catalog and repository-state inventory/classification/reporting;
- placement auditing and service-tier regeneration;
- encrypted-secret workflows;
- backup and storage analysis;
- hardlink analysis/application;
- media/content index analysis;
- drive health and migration procedures;
- development-tool and launcher installation.

Placement logic maps machine labels `homelab`, `staging`, and `deprecated` to tiers `production`, `staging`, and `deprecated`. The observed tier test maps deprecated containers to host `oci`; this is historical/deprecated placement classification, not active OCI ownership. `tests/test_placement_auditor.py` exercises drift classification, exit codes, output modes, unknown containers, and label edge cases without requiring live SSH.

### Fleet configuration and infrastructure as code

`ansible/` contains shared configuration plus playbooks for the development fleet, Mac mini, and general Homelab roles. Inventory and `group_vars` define target-specific values. Apply playbooks only after reading the target inventory and role contracts; machine names in documentation are not substitutes for inventory truth.

`infra/terraform` contains infrastructure-as-code. The Makefile exposes separate Tailscale and Cloudflare initialization, planning, and application targets, plus aggregate `tf-plan` and `tf-apply`. Terraform state/backend details are not present in the observed extract and must be read before mutation.

`config/developer-fleet.tsv`, `config/model-preference.yaml`, `config/lron-performer-aliases.json`, and `config/argus-budget-monitor.yaml` define fleet membership and supporting automation preferences. `config/llm-overview-tiers.json` supports generated LLM briefing tiers.

### Schedules and backups

Scheduling is split across:

- `crontabs/` for machine-specific crontabs and installation;
- `cron.d/homelab-lron-convert` for LRON conversion;
- `systemd-timers/` for backup and maintenance jobs.

Observed timers include primary and secondary main backups, Maya backups, ClamAV scans, and Czkawka scans. `config/maya-backup-paths.txt` scopes Maya backup inputs. `crontabs/no-auto-clone` documents or enforces the boundary against automatic repository cloning. Schedule presence does not prove installation or last-run success.

### Secrets

Encrypted environment files live under `secrets/`, including bundles for Arb, Atlas, backup snapshots, Clio, Convex, Float, Homelab, Maya, OpenClaw, and shared services. They are encrypted repository artifacts, not plaintext configuration sources.

The supported workflow is documented in `docs/SECRETS.md` and exposed through Make targets such as `gen-env`, `decrypt-secrets`, `encrypt-secrets`, `edit-secrets`, `setup-sops`, and `test-sops`. Never commit plaintext secret values, API keys, raw age identities, decrypted environment files, or generated credentials. Service-specific ownership is binding; notably Maya and Hermes credentials have separate owners.

### Homepage and operator UI

`homepage/` configures the Homepage dashboard through service, bookmark, settings, widget, Docker, Kubernetes, and Proxmox YAML files plus custom CSS/JavaScript. `homepage/custom.js` refreshes known Homelab and cloud-service state on a 30-second interval.

The JavaScript service lists are dashboard presentation configuration, not authoritative deployment inventory or health state. `services.yaml.backup-syntax-20260314-013809` is a backup artifact; `widgets.yaml.disabled` is disabled configuration. Do not treat either as active.

### Tests and verification boundaries

The observed test suite covers:

- Baywatch systemd hardening and deployment configuration;
- Maya orchestration deployment;
- incident consumer behavior;
- Atlas edge-worker contracts;
- Homelab CLI delegation and onboarding;
- service placement auditing;
- project catalog parity;
- generated service tiers.

These are static or local contract tests unless a test explicitly invokes a live service. They do not replace `GET /health`, incident polling, systemd status, Docker inspection, webhook delivery evidence, or backup-run evidence.

### Historical and non-authoritative material

`.archive/`, `docs/archive/`, `runs/`, `thoughts/`, timestamped backups, disabled files, and old placement records are retained evidence or working history. For example, `.archive/.sync-20260103/dashboard.py` contains an old Flask sync dashboard with shell execution; it is archived and must not be described, imported, deployed, or secured as part of the current control plane. Current contracts come from active source, configuration, tests, recent commits, and live probes.

## Canonical entry points

- [`AGENTS.md`](AGENTS.md) — ONE_SHOT v14 operator constitution, routing policy, worker vocabulary, and retired/disabled boundaries.
- [`README.md`](README.md) — human operating model, machine roles, fast paths, and deployment loop.
- [`homelab.yaml`](homelab.yaml) — normalized project identity, production placement declaration, monitoring state, observability sink, secret consumer, and documentation pointers.
- [`Makefile`](Makefile) — primary human command surface for Compose, deployment, audits, state inventory, secrets, Terraform, onboarding, documentation, and fleet operations.
- [`docker-compose.yml`](docker-compose.yml) — root tiered service composition and shared Docker network.
- [`services/`](services/) — service-specific Compose, configuration, and deployment assets.
- [`config/maya-orchestration-production.yml`](config/maya-orchestration-production.yml) — Maya production orchestration contract.
- [`config/baywatch-maya-orchestration-handoff.yml`](config/baywatch-maya-orchestration-handoff.yml) — Baywatch/Homelab-to-Maya handoff boundary.
- [`config/baywatch-production.example.yml`](config/baywatch-production.example.yml) — Baywatch production configuration example.
- [`docs/README.md`](docs/README.md) — active handbook index.
- [`docs/DOCUMENTATION_INDEX.md`](docs/DOCUMENTATION_INDEX.md) — complete documentation map.
- [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) — system structure and component relationships.
- [`docs/INFRA_STATE.md`](docs/INFRA_STATE.md) — declared infrastructure state.
- [`docs/OPERATIONS.md`](docs/OPERATIONS.md) — human fallback, service operations, and health procedures.
- [`docs/ROADMAP.md`](docs/ROADMAP.md) — remaining work and acceptance gates; verify recent commits before assuming an item is still pending.
- [`docs/HOMELAB_API.md`](docs/HOMELAB_API.md) — Homelab API, health, observation, incident, acknowledgement, and retired-route contracts.
- [`docs/SECRETS.md`](docs/SECRETS.md) — SOPS/age workflow and secret-handling rules.
- [`hestia/registry.yml`](hestia/registry.yml) and [`hestia/projects/`](hestia/projects/) — normalized project inventory and registrations.
- [`ansible/`](ansible/) — machine inventories, variables, roles, and fleet playbooks.
- [`infra/terraform`](infra/terraform) — Tailscale and Cloudflare infrastructure definitions.
- [`crontabs/`](crontabs/) and [`systemd-timers/`](systemd-timers/) — scheduled-job installation and unit definitions.
- [`scripts/`](scripts/) — CLI, deployment, audit, storage, backup, onboarding, and fleet automation.
- [`tests/`](tests/) — executable deployment and contract expectations.
- [`pyproject.toml`](pyproject.toml) — Python version, package metadata, development dependencies, and pytest configuration.
- [`llms.txt`](llms.txt) — concise machine-readable repository entry point.
- [`llms-full.txt`](llms-full.txt) — expanded selected context; subordinate to active source, `AGENTS.md`, and current operational evidence.
