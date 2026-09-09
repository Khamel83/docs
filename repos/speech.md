# LLM-OVERVIEW — speech
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
`speech` is a micro service project within the Homelab architecture (`github.com/Khamel83/speech`), currently maintained in `development` lifecycle state under `homelab` ownership.

## Machine & Host Ownership
No multi-host information is available; `homelab.yaml` configures production runtime host and process manager as `unknown` (`runtime.production.host: unknown`, `managed_by: unknown`).

## What is actually built
The codebase is currently a minimal service skeleton containing declarative Homelab contract metadata:
- **Homelab Contract (`homelab.yaml`)**: Implements `schema: homelab.project/v1` detailing project identity (`speech`, kind `service`, owner `homelab`), state (`standby`), and secret broker consumer ID (`speech`).
- **Runtime & Operations Configuration**: Specifies service unit `speech`, event sink `homelab-api:/observations` with `sanitized` evidence policy, and health source `homelab-api` (zero active checks).
- **Dependencies**: Explicitly specifies empty required and optional dependency lists (`required: []`, `optional: []`).
- **Source Code / Services**: No executable code, API routes, database schemas, or modules exist in the repository tree yet.

## Canonical entry points
- `homelab.yaml`: Primary homelab project contract for service identity, runtime, observability, and secrets integration.
- `README.md`: Referenced documentation index entry point in project contract.
- `docs/OPERATIONS.md`: Referenced operational documentation entry point in project contract.
