# LLM-OVERVIEW — openclaw
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
`openclaw` provides infrastructure configuration, deployment declarations, agent governance policies, and integration strategies for OpenClaw, ClawdBot, and MoltBot. It manages gateway runtime configurations, model provider routing (primary `glm-5-turbo` with local Ollama fallback), tool integrations (Brave search, voice transcription, file index search), and agent operations defined by the ONE_SHOT v13 framework.

## Machine & Host Ownership
- **Local Host / Homelab Node**: Host environment running the `openclaw-node` container service via `docker-compose.yml` (`image: openclaw:local`, UID/GID `1000:1000`), mounting local configuration paths (`~/github/openclaw`) and SQLite file index (`~/file-index/master-index.db`).
- **Mac Mini**: Configured host for the Ollama provider, serving local LLM fallback and heartbeat monitoring.
- Status probe output is disabled (`JANITOR_RUN_STATUS_PROBE=1` unset); `homelab.yaml` defines `runtime.production.host: unknown` with monitoring state `standby` under unit `openclaw`.

## What is actually built
- **Container Runtime Manifest (`docker-compose.yml`)**:
  - Service: `openclaw-node` built from `openclaw:local`.
  - Execution user: `1000:1000`.
  - Bind mounts: Read-only access to `~/github/openclaw` (`/workspace/openclaw-config`) and `~/file-index/master-index.db` (`/data/master-index.db`).
  - Environment: Receives `OPENCLAW_GATEWAY_TOKEN`.
  - Restart policy: `unless-stopped`.
- **Homelab Contract (`homelab.yaml`)**:
  - Specification: `homelab.project/v1` schema (ID `openclaw`, owner `homelab`, lifecycle `development`, kind `service`).
  - Health & Telemetry: `homelab-api` source with observations sent to `homelab-api:/observations` (sanitized evidence policy).
  - Operations reference: `README.md` (index) and `docs/OPERATIONS.md`.
- **Gateway Runtime Configuration (`openclaw.json`)**:
  - Primary model: `glm-5-turbo` (switched from `glm-4.7-flashx`).
  - Fallback & Heartbeat: Ollama provider running on Mac Mini.
  - Active capabilities: Voice transcription, Brave web search integration, memory search over master index database, and secrets hardening.
- **Agent Governance Framework (`AGENTS.md`)**:
  - ONE_SHOT v13 operator framework defining 3 operators:
    - `/short`: Rapid burn-down execution using standard defaults and git/context state.
    - `/full`: Structured project intake with `IMPLEMENTATION_CONTEXT.md` and checkpoints at 50% and 70%.
    - `/conduct`: Multi-model PMO orchestrator executing tasks across Claude, Codex, and Gemini.
  - 7 utility commands: `/handoff`, `/restore`, `/research`, `/freesearch`, `/doc`, `/vision`, `/secrets`.
  - Decision rules favoring simplest implementation, matching surrounding code patterns, and skipping non-blocking refactors.
- **Identity & Context Store**:
  - Core persona and tooling guidelines: `SOUL.md`, `TOOLS.md`, `USAGE.md`, `USER.md`, `HEARTBEAT.md`, `learnings.md`.
  - Integration strategies and reference docs: `docs/CONFIG.md`, `docs/OpenClaw Homelab Integration Strategy.md`, `docs/sessions/`.

## Canonical entry points
- **Service Deployment**: `docker compose up -d` (executes `openclaw-node` from repository directory).
- **Gateway Configuration**: `openclaw.json` (defines LLM routing, tokens, search, and transcription bindings).
- **Homelab Registry Spec**: `homelab.yaml` (homelab project definition contract).
- **Agent Execution**: `/short`, `/full`, `/conduct` CLI commands declared in `AGENTS.md`.
