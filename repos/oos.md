# LLM-OVERVIEW — oos
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
OOS (Organized Operational Setup) is a Python standard-library-first development workflow automation framework, systematic operational environment, and AI agent orchestration setup. It standardizes workflows across local machines and homelab infrastructure by combining CLI service launchers, ONE_SHOT v14 agent routing, Archon vault/API integration, RelayQ SSH node configuration, SystemD service management, and multi-provider LLM benchmarking.

## Machine & Host Ownership
- **Primary Linux Host (`ocivm-dev` / OCI VM)**: Local development root located at `/home/ubuntu/dev/oos`, managed as a SystemD unit `oos` (`homelab.yaml`).
- **Remote Worker Nodes (SSH via Tailscale)**:
  - `oci-ts`: Executes headless Codex sessions (`unset OPENAI_API_KEY && codex exec --sandbox danger-full-access`).
  - `macmini-ts`: Executes Gemini CLI tasks (`gemini "prompt"`).
- **Homelab External API Services**:
  - Argus Search API: Hosted at `http://100.112.130.100:8270` for web search resolution.
  - Archon Vault & API: Hosted at `https://archon.khamel.com` (configured via `ARCHON_URL`).

## What is actually built
- **Core Package & CLI Launcher (`pyproject.toml`, `run.py`, `Makefile`)**:
  - Python package `oos` (v1.2.0) built via `hatchling.build`. Runtime dependencies rely primarily on Python standard library and `python-dotenv` (optional extras for `mcp`, `flask` dashboard, `pytest`, `ruff`).
  - Interactive launcher `run.py` (`oos` entry point) executing mandatory development gate checks (`bin/dev-gate.sh`) and system status diagnostics (`lib.health_check`).
  - `Makefile` targets managing SystemD services (`install`, `start`, `stop`, `restart`, `status`), SystemD watchdog/resource tests (`test-watchdog`, `test-resources`), and POSIX shell linting (`shfmt`, `ShellCheck`).
- **ONE_SHOT v14 Agent Orchestration (`AGENTS.md`)**:
  - Operating contract specifying task classification (`plan`, `research`, `implement_small`, `implement_medium`, `test_write`, `review_diff`, `doc_draft`, `search_sweep`, `summarize_findings`, `janitor_*`) mapped across intelligence tiers (`glm_claude`, `codex`, `gemini_cli`, `free`, `claw_code`).
  - Command routing via `python3 -m core.router.resolve --class <class> --category <category>`.
  - Outbound search integration proxying through Argus (`http://100.112.130.100:8270/api/search` or MCP `mcp__argus__search_web`).
- **Archon API & Vault Secrets (`test_archon_api_integration.py`, `get_archon_secrets.py`, `auth.py`)**:
  - `ArchonAPI` Python client handling REST API authentication (`/api/login`), Bearer token retrieval, and secret fetching (`/api/secrets`) from Archon.
  - Helper functions for user authentication input validation (`auth.py`).
- **RelayQ Infrastructure & Remote Execution (`configure_ssh.py`)**:
  - Automated SSH configuration builder parsing `.env` parameters to generate node configuration manifests (e.g., node `ocivm-dev` on `localhost:22`).
- **LLM Benchmarking & Selection (`solo_creator_mecha_suit.py`, `benchmark_model_selector.py`)**:
  - Model selection framework evaluating value and quality metrics (e.g., `google/gemma-2-9b-it` for general research/planning vs `meta-llama/llama-3.1-70b-instruct` for critical coding).

## Canonical entry points
- `run.py` / `oos`: Interactive CLI launcher and dev-gate validation runner.
- `AGENTS.md`: Operating constitution governing ONE_SHOT task classification, routing rules, and SSH dispatch commands.
- `Makefile`: Commands for service lifecycle management, watchdog testing, and POSIX script validation.
- `test_archon_api_integration.py` / `get_archon_secrets.py`: Entry points for Archon API and vault secret integration.
- `configure_ssh.py`: Generator script for RelayQ SSH node topology.
- `solo_creator_mecha_suit.py`: Execution harness for model selection and LLM prompt dispatch.
