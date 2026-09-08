# LLM-OVERVIEW — Janus
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
Janus is the private central hub for cross-project work packets and task visibility across AI-assisted development sessions. It acts as a lightweight task/work-packet management layer, persisting project context and task progress in flat files (`state/` and `dashboard/`) while delegating infrastructure facts, secrets, live model credentials, and terminal model launchers (`cc`, `ccgo`, `cx`, `gem`, `oc`, `qwen`, `glm`, `kimi`, `mm25`, `mm27`) to `homelab`. Legacy OneShot execution behavior is retired and shut down in favor of Janus work packets.

## Machine & Host Ownership
According to `homelab.yaml` and local configuration, Janus runs as a client service (`unit: janus`, `lifecycle: development`, `owner: homelab`) on local developer environments; no multi-host information is available in local repo evidence.

## What is actually built
- **Core CLI Engine (`janus.py`)**: Python controller managing work packets, task statuses, and repository tracking (`do_work`, `do_tasks`, `do_status`, `repos`, `register`). Configured via environment variables `JANUS_HOME` (defaults to repo root) and `JANUS_GITHUB_ROOT` (defaults to `~/github`).
- **MCP Server (`mcp_server.py`, `bin/janus-mcp`)**: Stdio JSON-RPC 2.0 server wrapping core CLI capabilities. Exposes `janus_work`, `janus_tasks`, and `janus_status` tools to Claude Code and external AI orchestration layers.
- **Environment Setup Tooling (`bin/janus-setup.py`)**: Automated installer creating user-level symlinks in `~/.local/bin/` (`janus`, `janus-mcp`, `secrets`), installing slash commands in `~/.claude/commands/work.md`, and configuring MCP entries in `~/.claude/settings.json`.
- **Flat-File State Layer (`state/`)**: No-database persistence layer storing task lifecycle and project metadata:
  - `tasks.jsonl`: Work packet records and task state.
  - `events.jsonl`: Audit log of system events.
  - `projects.tsv`: Directory mappings for registered repositories.
  - `model-preference.json`: Model preference matrix for dispatch routing.
- **Task Dashboard (`dashboard/`)**: Static web interface (`index.html`) rendering task state sourced from `dashboard/tasks.jsonl` and `state/tasks.jsonl`.
- **Homelab Integration Contract (`homelab.yaml`)**: `homelab.project/v1` project manifest specifying operational metadata (`lifecycle: development`, `monitoring: standby`).
- **Verification Harness (`Makefile`)**: Operational smoke test (`make smoke`) verifying script compilation, temporary git repo setup, repository registration, task creation, and retrieval.

## Canonical entry points
- `janus work "<task>"`: Primary CLI command to create work packets, log tasks, and prepare handoffs.
- `python3 janus.py`: Direct script entry point supporting `work`, `tasks`, `status`, `repos`, and `register` commands.
- `python3 mcp_server.py` / `bin/janus-mcp`: Stdio JSON-RPC entry point exposing Janus tools for MCP-enabled client sessions.
- `python3 bin/janus-setup.py`: Environment configuration script registering system symlinks and Claude Code configurations.
- `make smoke`: Development verification command executing end-to-end task packet flows against isolated temporary git workspaces.
