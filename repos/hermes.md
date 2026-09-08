# LLM-OVERVIEW — hermes
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is

Hermes is the central orchestrator and interactive voice in Slack for the homelab AI architecture, coordinating between Maya (memory and context storage on port 8200) and Clio (task execution engine on port 8100). Hermes receives Slack input, triages user intents, dispatches structured task intake requests, updates and searches document context in Maya, runs scheduled fleet context pushes across repositories, and exposes typed client capabilities over an internal Model Context Protocol (MCP) server interface.

The codebase is implemented in Python 3.12+ using FastAPI, Uvicorn, HTTPX (async), `slack-bolt` (Socket Mode), and the official Python MCP SDK, with package management driven by `uv`.

## Machine & Host Ownership

Based on `AGENTS.md` (with status probe output disabled), Hermes runs in the Mac Mini Homelab environment (`omars-mac-mini`) over Tailscale. The internal MCP server (`hermes-internal`) runs on the Mac mini host under `launchd` supervision (`com.khamel.hermes-mcp-server.plist`) listening on port 8643 (`http://omars-mac-mini:8643/mcp`). It interacts with Maya (`http://...:8200`) and Clio (`http://...:8100`) via HTTP APIs over Tailscale using bearer token authentication.

## What is actually built

### 1. Triage & Orchestration Engine (`src/hermes/triage.py`, `src/hermes/orchestrator.py`)
- **`SilentTriageEngine`**: Classifies incoming Slack messages into discrete action categories: `KEEP` (file and document ingestion to Maya), `DO` (task submission to Clio structured intake), `ANSWER_STATUS` (retrieving Clio execution status), or `CLARIFY`.
- **`HermesOrchestrator`**: Central message pipeline (`handle_slack_message`). Executes `MayaClient.ingest_file` for media attached to `KEEP` messages, calls `ClioClient.submit_intake` (`POST /api/intake`) for `DO` actions, and queries `ClioClient.get_status` for status inquiries, formatting and returning results to origin Slack threads.

### 2. FastAPI Core & Webhook Service (`src/hermes/app.py`, `src/hermes/webhooks.py`, `src/hermes/config.py`)
- **FastAPI Application**: Factory (`app.py`) providing `/` root metadata and `/health` health check endpoints on `HERMES_SERVICE_PORT` (default 3000).
- **Webhooks Listener (`POST /webhook/clio_event`)**: Bearer-token protected endpoint (`CLIO_INTERNAL_TOKEN` or `HERMES_CLIO_EVENT_TOKEN`) processing Clio lane lifecycle events (lane A/B/D execution status), updating Slack threads, and keeping context updated.
- **Pydantic Configuration (`config.py`)**: `pydantic-settings` model managing configuration for Maya/Clio endpoints, bearer tokens (`MAYA_INGEST_TOKEN`, `CLIO_INTERNAL_TOKEN`, `HERMES_MCP_TOKEN`, `ARGUS_API_KEY`), channel IDs, and user allowlists.

### 3. Slack Bridge & Router (`src/hermes/slack_bridge.py`, `model-router.py`)
- **Socket-Mode Bridge (`slack-bolt`)**: Listens to real-time Slack socket events, filters out bot messages and edits, validates user permissions against allowlists, extracts file attachments as base64 strings, and passes structured `SlackMessage` payloads to `HermesOrchestrator`.
- **`model-router.py`**: Model router supporting ChatGPT Luna as the default model, tool-only routing logic, watchdog probe support, and container environment setup.

### 4. Typed Client Libraries (`src/hermes/clients/maya.py`, `src/hermes/clients/clio.py`)
- **`MayaClient`**: Async HTTP client covering Maya file ingestion (`ingest_file`), vault note appending (`POST /notes/append`), file metadata operations (`list_files`, `get_file`), and semantic file search (`search`).
- **`ClioClient`**: Async HTTP client wrapping Clio task dispatch via structured intake (`submit_intake`), execution status (`get_status`), project enumeration (`list_projects`, supporting array payloads), and context pack batch updates (`batch_upsert_context_packs`).

### 5. Context Store (`src/hermes/context.py`)
- **`HermesContextStore`**: In-memory conversation turn tracker and project-level context map serializer injected into the LLM context window.

### 6. Internal MCP Server (`hermes-internal`)
- **9 MCP Tools Engine**: Built on the official MCP Python SDK, exposing 9 tools wrapping typed Maya and Clio client methods (file searches, vault updates, project listing, intake submission, context pack upserts) over port 8643 (`http://omars-mac-mini:8643/mcp`).
- **Supervisor & Auth**: Protected via `Authorization: Bearer ${HERMES_MCP_TOKEN}` header verification and managed via launchd (`scripts/com.khamel.hermes-mcp-server.plist`).

### 7. Fleet Context Push Job (`hermes-push-fleet-context`)
- **Batch Context Synchronizer**: Scheduled task running every 4 hours (`uv run hermes-push-fleet-context`). Pulls Maya fleet context for all registered repositories and pushes batch updates to Clio's `POST /api/context-packs/batch` endpoint via `ClioClient.batch_upsert_context_packs()`, populating Clio's `FleetContextSnapshot` table.

## Canonical entry points

- **FastAPI Service**: `uv run uvicorn hermes.app:app --port 3000` (or `uvicorn hermes.app:app`).
- **Fleet Context Synchronizer**: `uv run hermes-push-fleet-context` (CLI entry point for 4-hour batch context push job).
- **Internal MCP Server**: `launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.khamel.hermes-mcp-server.plist` (runs on port 8643).
- **Slack Router / Bridge**: `python model-router.py` / `uv run python src/hermes/slack_bridge.py`.
- **Test Suite**: `uv run pytest` (executes unit tests for triage, client APIs, webhooks, and routing).
