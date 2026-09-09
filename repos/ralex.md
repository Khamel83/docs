# LLM-OVERVIEW — ralex
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
Ralex (RalexOS) is an integration suite for Anthropic's Claude Code CLI, Y-Router / Claude Code Router (CCR), OpenRouter AI models, and Model Context Protocol (MCP) servers. It provides multi-model CLI routing across 10 OpenRouter endpoints (including GPT-5 Nano, Kimi K2, Qwen3 Coder, and Gemini Flash) with in-session model switching and tool execution across 22 MCP servers. The repository is preserved in standby status under homelab management (`homelab.yaml`), containing active V1 routing shell scripts alongside archived legacy Agent OS / k83 framework components.

## Machine & Host Ownership
Configured in `homelab.yaml` for production deployment on host `mba` (`managed_by: unknown`, unit `ralex`, `sync_mode: pull-on-demand`) under `homelab` ownership in the `development` lifecycle phase with monitoring state `standby`. Operational telemetry and health checks report to `homelab-api` with events emitted to `homelab-api:/observations`.

## What is actually built
- **Router & CLI Integration Layer:** Shell scripts (`setup-y-router.sh`, `setup-claude-code-router.sh`) automate proxy configuration and router setup for Y-Router and Claude Code Router. Router configuration maps through `config/ccr.config.template.json` to generate `config-router.json`. `claude-functions.sh` supplies shell environment functions enabling model switching commands (`/model kimi-k2`, `/model qwen3-coder`) and CLI wrappers.
- **MCP Server Integration Ecosystem:** Pre-configured support for 22 MCP servers, including filesystem management (`filesystem`), persistent conversation tracking (`memory-bank`), step-by-step reasoning (`sequential-thinking`), repository integration (`github-integration`), automated browser testing (`playwright-testing`), error monitoring (`sentry-monitoring`), and agent orchestrations (`mcp-orchestrator`, `k83-framework`).
- **Validation & Test Automation:** `ralexos-complete.sh` provisions the complete environment; `test-ralexos.sh` and `run-comprehensive-tests.sh` execute automated verification suites for model routing and tool calling. `demo.sh` provides an interactive demonstration entry point. `config/zen-test-with-mcp.json` and `config/redis.conf` supply test harness specs and optional Redis caching configuration.
- **Archived Agent OS Subsystems (`Old Ralex/`):** Python modules from the historical `ralex-integration-package`, including `state_manager.py` (SQLite and file-based multi-tool conversation context persistence), `context_analyzer.py` (Git and Agent OS project state inspector), `methodology_engine.py` (three-phase task decomposition engine), `buildkite-mcp` integration, prompt comparison benchmarks (`comparison_prompt.txt`), and model response benchmarks (`claude_code_response.js`, `deepseek_v3_response.txt`).
- **Legacy Artifacts & Distributions (`archive/`):** Preserved distribution bundles (`ralexos-dist.zip`, `ralexos.sh`), installation automation (`integrate-opencode.sh`, `package-dist.sh`), prompt test matrices (`test-prompts.json`, `zen-test.json`), and legacy test run outputs (`comprehensive_test_20250812_205810` containing sample Flask applications).

## Canonical entry points
- `setup-y-router.sh`: Primary setup script configuring Y-Router proxy and OpenRouter API integration for Claude Code.
- `setup-claude-code-router.sh`: Configuration script for Claude Code Router (CCR) using template settings from `config/ccr.config.template.json`.
- `ralexos-complete.sh`: Provisioning entry point for instantiating the complete router, MCP servers, and shell integration environment.
- `claude-functions.sh`: Sourceable shell script defining helper commands and model switching macros.
- `test-ralexos.sh`: Test script for validating model connectivity and router execution.
- `run-comprehensive-tests.sh`: Test suite runner for multi-model integration and MCP server verification.
- `demo.sh`: Interactive demonstration script for verifying active terminal session capabilities.
