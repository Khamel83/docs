# LLM-OVERVIEW — servers
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
Reference implementation servers for the Model Context Protocol (MCP), maintained by the MCP steering group (LF Projects, LLC). Contains a collection of Node.js / TypeScript ES module (`type: "module"`) workspaces under `src/` managed by the `@modelcontextprotocol/servers` monorepo package (v0.6.2). Serves as educational reference code demonstrating MCP tools, resources, and transport capabilities.

## Machine & Host Ownership
No multi-host or active production deployment details are available; `homelab.yaml` specifies `runtime.production.host: unknown` (`managed_by: unknown`, `unit: servers`) in `standby` monitoring state, and status probe output is disabled.

## What is actually built
- **Workspace Monorepo Architecture**: Monorepo managed via npm workspaces (`src/*`). Built for Node.js using TypeScript (`tsconfig.json`) in ES module mode.
- **Server Implementations (`src/`)**:
  - `src/everything`: Multi-transport reference server exposing a CLI launcher (`index.js`) for `stdio`, `sse`, and `streamableHttp` transports (`transports/stdio.js`, `transports/sse.js`, `transports/streamableHttp.js`).
  - `src/memory`: Knowledge graph persistence server storing entity-relation graphs in line-delimited JSON (`memory.jsonl`). `KnowledgeGraphManager` provides entity (`Entity`) and relation (`Relation`) creation and query tools. Includes automatic migration from legacy `memory.json` via `ensureMemoryFilePath` and `MEMORY_FILE_PATH` environment override. Tested with Vitest (`vitest.config.ts`, `v8` coverage).
  - `src/filesystem`: Filesystem MCP server providing safe file operations with MacOS symlink resolution, Windows drive letter root normalization (`fileURLToPath`), path traversal protection, and destructive operation hints.
  - `src/fetch`: HTTP fetch server with proxy handling (`httpx`), Python `uv` lockfile management, and robust malformed input error handling.
  - `src/git`: Git repository inspection and mutation server with argument injection guards across `git_show`, `git_create_branch`, `git_log`, and `git_branch`.
  - `src/sequentialthinking`: Dynamic problem-solving tool using structured sequential thinking steps with `z.coerce` parameter type coercions and MCP tool annotations.
  - `src/time`: Time lookup and timezone conversion tools (`get_current_time`, `convert_time`) with tool annotations.
- **Release Automation**: `scripts/release.py` executable Python script (`uv run`, Python >=3.12) using `click` and `tomlkit` for workspace versioning and git/release operations.

## Canonical entry points
- `npm run build`: Compiles all package workspaces (`npm run build --workspaces`).
- `npm run watch`: Incremental TypeScript watcher across workspaces (`npm run watch --workspaces`).
- `npm run link-all`: Links all workspace packages locally (`npm link --workspaces`).
- `npm run publish-all`: Publishes workspace packages to the public npm registry (`npm publish --workspaces --access public`).
- `node src/everything/index.js [stdio|sse|streamableHttp]`: Launcher for the everything server with chosen transport.
- `node src/memory/index.js`: Memory server startup entry point.
- `scripts/release.py`: Release automation script.
