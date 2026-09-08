# LLM-OVERVIEW — MJH
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
MJH (`mjh`) is a Homelab service repository in early development (`lifecycle: development`) owned by `homelab`. It defines project metadata and infrastructure contract declarations adhering to the `homelab.project/v1` schema.

## Machine & Host Ownership
No multi-host information is available; runtime configuration in `homelab.yaml` lists production host and manager as `unknown` (unit: `mjh`).

## What is actually built
The repository contains project configuration and minimal documentation:
- **Project Contract (`homelab.yaml`)**: Defines project identity (`id: mjh`, `name: Mjh`, repository `https://github.com/Khamel83/MJH`), kind (`service`), lifecycle (`development`), and owner (`homelab`). Sets monitoring state to `standby`, health check source to `homelab-api` (empty checks), observability event sink to `homelab-api:/observations` (`sanitized` policy), secret consumer `mjh`, and runtime systemd unit `mjh`. Required and optional dependencies are empty.
- **Documentation (`README.md`)**: Contains project title header and initial placeholder test content.
- **Source Code**: No application source code, API routes, data models, or executable entry points are implemented yet.

## Canonical entry points
- `homelab.yaml`: Service definition, observability settings, and homelab project contract.
- `README.md`: Primary repository documentation index.
