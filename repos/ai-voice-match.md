# LLM-OVERVIEW — ai-voice-match
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
`ai-voice-match` is a service repository designed to generate AI responses matching an authentic personal voice using prompt engineering without model training. It is owned by `homelab` under the `development` lifecycle state with monitoring configured in `standby` mode.

## Machine & Host Ownership
No multi-host information is available in the repository configurations or status probe (`runtime.production.host` is `unknown`).

## What is actually built
The repository consists of project metadata and documentation scaffolding without active application source code, executable runtime modules, or database tables:
- **Project Infrastructure Contract (`homelab.yaml`)**: Implements schema `homelab.project/v1`. Configures service identifier `ai-voice-match`, production systemd unit `ai-voice-match`, health reporting via `homelab-api`, and telemetry logging to `homelab-api:/observations` with sanitized evidence policies. Required and optional dependencies, repair IDs, and secret references are currently empty lists.
- **Documentation Index (`README.md`)**: Serves as the primary documentation entry point referenced by `homelab.yaml`.

## Canonical entry points
- `homelab.yaml`: Homelab project definition, runtime service unit target, health check spec, and telemetry configuration.
- `README.md`: Project documentation index referenced by the `homelab.yaml` docs spec.
