# LLM-OVERVIEW — secrets-vault
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
Deprecated repository formerly serving as a secrets vault and agent skill store for homelab services. Active development, secrets storage, and Claude Code skills have been migrated and consolidated into `oneshot` (`https://github.com/Khamel83/oneshot`). The repository is inactive and designated for archiving.

## Machine & Host Ownership
There is no multi-host information available.

## What is actually built
The repository contains no executable source code, active API endpoints, or database schemas. It consists of project contract specifications, deprecated documentation, and legacy SOPS-encrypted configuration files:

- **Project Specification (`homelab.yaml`)**:
  - Schema: `homelab.project/v1`
  - Metadata: `project.id: secrets-vault`, `name: Secrets Vault`, `kind: service`, `lifecycle: development`, `owner: homelab`
  - Runtime & Health: `monitoring.state: standby`, `runtime.production.host: unknown`, `managed_by: unknown`, `unit: secrets-vault`, `health.source: homelab-api` (no checks defined)
  - Observability: `observability.event_sink: homelab-api:/observations`, `evidence_policy: sanitized`
  - Dependencies: `required: []`, `optional: []`
  - Secrets Consumer: `secrets.broker_consumer: secrets-vault`

- **Secret Storage Artifacts**:
  - `secrets.yaml`, `secrets.env.encrypted`: Legacy SOPS-encrypted stores containing Cloudflare API keys, homelab service credentials, and OCI-Dev infrastructure secrets (migrated to `oneshot/secrets/`).
  - `homelab-secrets.env.encrypted`, `homelab-secrets.yaml.encrypted`, `homelab.env.encrypted`: Additional encrypted environment variable stores for homelab components.

- **Deprecation & Skill Definitions**:
  - `README.md`: Deprecation notice citing migration to `oneshot` (`oneshot/.claude/skills/`, `oneshot/secrets/`, `oneshot/README.md`) with instructions to archive the repository.
  - `SKILLS.md`, `CLAUDE.md`, `HOW.md`: Historic skill definitions (e.g., `BUILD_WITH_SKILLS` algorithm, OCI-Dev deployment) removed via commit `d369b9c` and relocated to `oneshot`.

## Canonical entry points
- `README.md`: Primary reference containing deprecation notice and repository consolidation targets.
- `homelab.yaml`: Homelab project lifecycle contract and observability specification.
- `secrets.yaml` / `secrets.env.encrypted`: Legacy encrypted secret payload stores.
- `SKILLS.md`: Historical agent skill definitions (deprecated).
