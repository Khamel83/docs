# LLM-OVERVIEW — hermes-final-live-orchestration
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is

Hermes is the central service-layer orchestration engine and Slack-facing personal assistant router in the homelab ecosystem. It acts as the conversational interface and decision-maker in Slack, coordinating state and task execution between Maya (port 8200, memory and context store) and Clio (port 8100, task execution engine). Hermes processes Slack events via Socket Mode, triages user intent into memory operations (`KEEP`), task dispatches (`DO`), status queries (`ANSWER_STATUS`), or user clarifications, routes LLM inference to `gpt-5.6-luna` (primary via `openai-codex`) with `opencode-go/minimax-m3` and `deepseek-v4-flash` fallbacks, exposes a 9-tool MCP server on port 8643 for cross-service tooling, and executes periodic fleet context pushes.

## Machine & Host Ownership

- **`homelab` host** (Linux/Docker runtime): Runs the main Hermes service container (`services/hermes/docker-compose.yml` deployed via `scripts/hermes-bootstrap.sh`), hosting the HTTP orchestrator (`src/hermes/app.py`, port 3000 / health probe 8642), Socket-Mode Slack bridge (`src/hermes/slack_bridge.py`), and standalone model router (`model-router.py`, port 8080).
- **`omars-mac-mini` / Mac Mini host** (Homelab node connected over Tailscale):
  - Target Maya service endpoint on port 8200 (`http://100.113.216.27:8200`).
  - Target Clio service endpoint on port 8100 (`http://100.113.216.27:8100`).
  - `hermes-internal` MCP server running on port 8643 (`http://omars-mac-mini:8643/mcp`) managed via launchd service `com.khamel.hermes-mcp-server.plist`.
  - `hermes-watchdog` monitoring daemon managed via launchd service `com.khamel.hermes-watchdog.plist`.

## What is actually built

- **Core Orchestration Package (`src/hermes/`)**:
  - `app.py`: FastAPI application exposing health routes (`/`, `/health`), webhook handlers, and the MCP server launch function (`main_mcp`). Exposed via console script `hermes-service`.
  - `config.py`: Pydantic-settings module defining Tailscale endpoints for Maya (`http://100.113.216.27:8200`, `MAYA_INGEST_TOKEN`) and Clio (`http://100.113.216.27:8100`, `CLIO_INTERNAL_TOKEN`), Slack tokens, allowlisted users, MCP auth (`HERMES_MCP_TOKEN`), and channel routing policies (e.g. divorce channel, live Maya channel `C0BGY8YQU5N`).
  - `triage.py`: `SilentTriageEngine` classifying `SlackMessage` events into `KEEP` (Maya note/file ingest), `DO` (Clio work intake submission), `ANSWER_STATUS` (Clio status format and response), or `CLARIFY`.
  - `orchestrator.py`: `HermesOrchestrator` handling message intake, executing client calls based on triage classification, and dispatching thread replies.
  - `slack_bridge.py`: `SlackBridge` built on `slack-bolt` (Socket Mode). Filters bot events and non-allowlisted users, normalizes attachment payload formats, deduplicates messages, and routes events to `HermesOrchestrator`. Exposed via console script `hermes-slack-bridge`.
  - `webhooks.py`: FastAPI router handling `POST /webhook/clio_event` authenticated via `CLIO_INTERNAL_TOKEN` or `HERMES_CLIO_EVENT_TOKEN` for Clio lifecycle event (Lanes A/B/D) updates.
  - `context.py`: `HermesContextStore` managing in-memory chat turns and per-project prompt context formatting.
  - `fleet_context_push.py`: Scheduled task running every 4 hours (`hermes-push-fleet-context`) fetching Maya fleet context packs (`GET /fleet/context/{slug}`) and batching them to Clio (`POST /api/context-packs/batch`).
  - `mcp_server.py`: MCP server implementation using the Python `mcp` SDK. Exposes 9 tools over streamable HTTP transport on port 8643 (`/mcp`) covering Maya (`ingest_file`, `note_append`, `request_context`, `request_create`) and Clio (`submit_intake`, `get_status`, `list_projects`, engine tools). Exposed via console script `hermes-mcp-server`.
  - `clients/maya.py`: `MayaClient` HTTP client for Maya (`ingest_file()`, `append_note()`, `list_files()`, `get_file()`, `search()`).
  - `clients/clio.py`: `ClioClient` HTTP client for Clio (`submit_intake()` via structured `POST /api/intake`, `get_status()`, `list_projects()` returning bare project array).
- **Model Router & Adapters (`model-router.py`, `overrides/gateway/platforms/slack.py`)**:
  - `model-router.py`: Standalone HTTP router proxying `deepseek-v4-flash` (default chat completions) and `opencode-go/minimax-m3` (escalation Anthropic messages API) via OpenCode Go.
  - Primary default model policy: `gpt-5.6-luna` via authenticated `openai-codex` with `xhigh` reasoning.
- **Service Scripts & Configuration (`scripts/`, `config/`, `homelab.yaml`)**:
  - `scripts/com.khamel.hermes-mcp-server.plist`: Launchd service plist running `hermes-mcp-server` on Mac Mini with logs directed to `~/Library/Logs/hermes-mcp-server/`.
  - `scripts/com.khamel.hermes-watchdog.plist` & `scripts/hermes_watchdog.py`: Launchd health watchdog daemon.
  - `scripts/hermes-bootstrap.sh`: Homelab container deployment bootstrap.
  - `scripts/hermes-slack-discover-channels.py`: CLI tool for Slack channel discovery.
  - `scripts/config-drift-check.sh`: Config template validator.
  - `homelab.yaml`: Project metadata specification (`hermes`, service lifecycle development, observing monitoring state).
- **Dependencies & Tests (`pyproject.toml`, `tests/`)**:
  - Python >=3.12 managed via `uv`. Core dependencies: `fastapi`, `httpx`, `pydantic-settings`, `uvicorn`, `slack-bolt`, `slack-sdk`, `aiohttp`, `mcp`.
  - Pytest suite covering `app`, `clients`, `config`, `context`, `dispatch_context`, `fleet_context_push`, `hermes_model_config`, `hermes_watchdog`, `maya_channel_config`, `mcp_server`, `slack_bridge`, and `triage`.

## Canonical entry points

- **CLI Commands (Package Console Scripts)**:
  - `uv run hermes-service`: Starts the FastAPI orchestrator service (`hermes.app:main`).
  - `uv run hermes-slack-bridge`: Launches the Socket-Mode Slack bridge (`hermes.slack_bridge:main`).
  - `uv run hermes-push-fleet-context`: Executes batch fleet context sync from Maya to Clio (`hermes.fleet_context_push:main`).
  - `uv run hermes-mcp-server`: Launches the MCP server on port 8643 (`hermes.app:main_mcp`).
- **HTTP & Transport Endpoints**:
  - `http://<host>:3000/` & `http://<host>:3000/health`: Health endpoints for the primary service.
  - `http://<host>:3000/webhook/clio_event`: Callback receiver for Clio event notifications.
  - `http://omars-mac-mini:8643/mcp`: MCP tool server endpoint (requires `HERMES_MCP_TOKEN`).
  - `http://<host>:8080/v1/chat/completions`: Model router proxy endpoint (`model-router.py`).
- **System Administration & Process Control**:
  - `launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.khamel.hermes-mcp-server.plist`: Manages Mac Mini MCP server daemon.
  - `launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.khamel.hermes-watchdog.plist`: Manages Mac Mini watchdog process.
  - `bash scripts/hermes-bootstrap.sh`: Triggers homelab service deployment.
