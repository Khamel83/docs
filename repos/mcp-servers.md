# LLM-OVERVIEW — mcp-servers
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
Multi-language monorepo containing Model Context Protocol (MCP) servers, client integration libraries, agent orchestration frameworks, and tooling. Includes Python agent components (`mcp-agent`, `deepview_mcp`, `mcp_scheduler`), Go-based MCP implementations (`buildkite-mcp-server`, `github-mcp-server`), Node.js/TypeScript servers (`@vectorize-io/vectorize-mcp-server`, Playwright MCP integration), and container/helm deployment configurations. Registered as a homelab service via `homelab.yaml`.

## Machine & Host Ownership
No multi-host deployment information is available (homelab.yaml specifies production host as `unknown` and status probe output is unavailable).

## What is actually built
- **Homelab Service Spec (`homelab.yaml`)**: Declares project `mcp-servers` (`homelab.project/v1`, owner `homelab`, lifecycle `development`, standby monitoring linked to `homelab-api`).
- **Python Subsystem (`mcp-agent`, `deepview_mcp`, `mcp_scheduler`)**:
  - `pyproject.toml`: Defines package `mcp-agent` (v0.1.8) managed via `uv`. Core dependencies include `mcp` (>=1.10.1), `fastapi`, `instructor`, `pydantic` (>=2.10.4), `opentelemetry-*`, `aiohttp`, `websockets`, `rich`, `typer`, `numpy`, `scikit-learn`, and `prompt-toolkit`. Optional extras target Anthropic (`bedrock`, `vertex`), OpenAI, Azure, Google GenAI, Cohere, and Temporal.
  - `deepview_mcp/`: Contains Python MCP server logic (`server.py`), CLI driver (`cli.py`), package init (`__init__.py`), and tests (`test.py`).
  - `mcp_scheduler`: Modules for SSE transport tool serving (`test_well_known.py`) and job utilities (`test_utils.py` for duration formatting and cron parsing).
  - Utility scripts: `scripts/format.py`, `scripts/lint.py`, `scripts/gen_schema.py`, `scripts/promptify.py`, and `compress.py`.
- **Go Subsystem (`buildkite-mcp-server`, `cmd/`)**:
  - `go.mod` (Go 1.24.5): Builds `github.com/buildkite/buildkite-mcp-server` using `mark3labs/mcp-go` (v0.36.0), `buildkite/go-buildkite/v4`, `buildkite-logs`, `alecthomas/kong`, `zerolog`, `terminal-to-html`, and OpenTelemetry OTLP gRPC instrumentation.
  - `cmd/`: Command binaries for `github-mcp-server` and `mcpcurl`.
- **Node.js / TypeScript Subsystem**:
  - `package.json`: Defines `@vectorize-io/vectorize-mcp-server` (v0.4.3) with entry binary `dist/index.js`, using `@modelcontextprotocol/sdk` (^1.4.1), `@vectorize-io/vectorize-client`, `dotenv`, and `p-queue`.
  - `config.d.ts`: Configures Playwright browser integration and tool capabilities (`core`, `core-tabs`, `core-install`, `vision`, `pdf`).
  - `setup-claude-server.js`: Server setup script with Google Analytics event reporting and crypto random UUID generation.
  - `index.js`, `cli.js`: Root JS execution entry points.
  - `vitest.workspace.ts`: Configures Vitest workspace packages (`packages/*`, `apps/*`).
- **MCP Server Modules & Commands**:
  - Dedicated integrations in `buildkite-mcp/`, `circleci-mcp/`, `deepview-mcp/`, `chart/semgrep-mcp` (Helm chart), `agent-os/`, and `claude-code/agents`.
  - Workflow command specifications in `commands/`: `analyze-product.md`, `create-spec.md`, `execute-tasks.md`, `plan-product.md`.

## Canonical entry points
- **Node.js Commands**:
  - `npm run build`: `tsc && node -e "require('fs').chmodSync('dist/index.js', '755')"`
  - `npm run dev`: `npm run build && npx @modelcontextprotocol/inspector node dist/index.js`
  - `npm run lint`: `eslint src/**/*.ts`
  - `npm run lint:fix`: `eslint src/**/*.ts --fix`
  - `npm run format`: `prettier --write .`
- **Python & Makefile Commands**:
  - `make sync`: `uv sync --all-extras --all-packages --group dev`
  - `make format`: `uv run scripts/format.py`
  - `make lint`: `uv run scripts/lint.py --fix`
  - `make tests`: `uv run pytest`
  - `make coverage`: `uv run coverage run -m pytest`
  - `make schema`: `uv run scripts/gen_schema.py`
  - `make prompt`: `uv run scripts/promptify.py`
- **Runtime Executables**:
  - Python: `deepview_mcp/server.py`, `deepview_mcp/cli.py`
  - Node.js: `index.js`, `cli.js`, `setup-claude-server.js`, `dist/index.js`
  - Go: `cmd/github-mcp-server`, `cmd/mcpcurl`, `buildkite-mcp/`
  - Infrastructure: `homelab.yaml`, `Dockerfile`, `Dockerfile.local`, `compose.yaml`
