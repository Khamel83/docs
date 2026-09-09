# LLM-OVERVIEW — openclaw-config
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
`openclaw-config` is the configuration, agent persona definition, workspace environment, and service integration repository for the OpenClaw AI gateway ecosystem. It contains system-level gateway configurations (`openclaw.json`), execution security rules (`exec-approvals.json`), multi-agent workspace directories (`workspace-*`), persona definitions (`agents/`), Telegram command verification hashes (`telegram/`), an inbound iMessage webhook service (`inbound-imessage.py`), and homelab service manifest contracts (`homelab.yaml`).

## Machine & Host Ownership
Configuration in `homelab.yaml` defines the production host and manager as `unknown` with monitoring state set to `standby`. Local service components (`inbound-imessage.py`) default to listening on `127.0.0.1` (localhost). No multi-host deployment topology or explicit server assignments are defined in available configuration files or status probes.

## What is actually built
- **OpenClaw Gateway Core & Security Policy**:
  - `openclaw.json`: Central runtime configuration file for the OpenClaw gateway.
  - `exec-approvals.json`: Execution security policy defining approved commands and agent permissions.
  - `start-gateway.sh`: Shell script entry point to launch the OpenClaw gateway service.
  - `openai-access-token`: Authentication token store for OpenAI API access.
  - `update-check.json`: State file for tracking gateway updates and version checks.

- **Agent Personas & Workspaces**:
  - Agent personas (`agents/`): Configured profiles for `eagle`, `forge`, `main`, `marcus`, `talon`, and `trojan`.
  - Workspaces (`workspace-eagle/`, `workspace-forge/`, `workspace-marcus/`, `workspace-talon/`, `workspace-trojan/`, `workspace-zeno/`): Isolated workspace directories containing persona context markdown files (`AGENTS.md`, `GEMINI.md`, `HEARTBEAT.md`, `IDENTITY.md`, `SOUL.md`, `TOOLS.md`, `USER.md`) and persistent operational state directories (e.g., `data/`, `argus/`).

- **Inbound iMessage Integration (`inbound-imessage.py`)**:
  - FastAPI server acting as an HTTP endpoint for messages forwarded from BlueBubbles webhooks or `imessage-poller`.
  - Binds to `127.0.0.1:8889` by default (configurable via `LISTEN_HOST` and `LISTEN_PORT`).
  - Enforces Bearer token authentication via `OPENCLAW_TOKEN`.
  - Appends incoming iMessage payloads into the Marcus workspace directory (`~/.openclaw/workspace-marcus`) for downstream execution.
  - Dependencies: FastAPI, Uvicorn, standard Python libraries (`os`, `re`, `sys`, `subprocess`, `pathlib`, `datetime`, `logging`).

- **Telegram Bot Command Verification (`telegram/`)**:
  - Command hash text signatures (`command-hash-*.txt`) for validating command integrity across agents (`default`, `eagle`, `forge`, `main`, `marcus`, `talon`, `trojan`, `zeno`).
  - `update-offset-default.json`: Tracking file for Telegram API message offsets.

- **Homelab Service Contract (`homelab.yaml`)**:
  - Declarative manifest (`homelab.project/v1`) identifying `openclaw-config` as a service in development.
  - Configures standby monitoring and specifies telemetry event reporting to `homelab-api:/observations`.

## Canonical entry points
- `start-gateway.sh`: Startup script for the main OpenClaw gateway runtime.
- `inbound-imessage.py`: FastAPI server for processing incoming iMessage webhooks on port 8889.
- `openclaw.json`: Primary configuration file for gateway settings.
- `exec-approvals.json`: Security rules for agent execution approvals.
- `homelab.yaml`: Homelab project definition, monitoring, and observability contract.
