# LLM-OVERVIEW — docs-cache
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
A centralized local documentation cache and tooling repository for AI agents in the homelab environment. It provides structured, local access to external library and service documentation to avoid unnecessary web searches, enforcing a strict search priority (Docs Cache → `/freesearch` Exa API → Training Data → WebSearch). It includes a curated documentation cache tree, a catalog of 514 trackable libraries, and the `docs-link` CLI executable for symlinking cached documentation directly into consuming project repositories.

## Machine & Host Ownership
No specific host machine deployment information is configured (`homelab.yaml` lists production host as `unknown`, status probe is disabled, and no `AGENTS.md` is present).

## What is actually built
- **Documentation Cache Subsystem (`docs/cache/`)**:
  - Organized into four primary category trees: `tools/`, `python/`, `javascript/`, and `services/`.
  - Maintained via `docs/cache/.index.md`, a Markdown catalog indexing 55+ cached libraries (+14 homelab-specific entries) structured with `Name`, `Category`, `Related`, `URL`, and `Cached` timestamp.
  - Subsystem breakdown:
    - `tools/`: Anthropic ecosystem (SDKs for Python, TypeScript, Go; Claude Code; Cookbooks; Quickstarts; Anthropic Tools; Agent Skills), OpenAI, LiteLLM, MCP, Redis, aiohttp, Typer, n8n, Home Assistant, Pandas, Convex, Supabase, Vercel, Traefik, Sentry, Cloudflare (Workers, Tunnel, DNS), Nginx, Terraform, SOPS/Age, MergerFS, Pi-hole, Jellyfin, Systemd, Tailscale (Funnel, ACL). Supports topic annotations under `annotations/`.
    - `python/`: BeautifulSoup4, Pydantic, FastAPI, TikToken, Transformers, Uvicorn, Click, HTTPX, Poetry, Requests, Pytest.
    - `javascript/`: Astro, BetterAuth, React, Playwright, TypeScript, Vite, Next.js, Hono, Zod, TailwindCSS.
    - `services/`: Docker, Docker-Compose, Tailscale, Cloudflare, Polymarket.
  - `docs/external/`: Local target directory for linked doc symlinks (`convex`, `polymarket`).
- **Documentation Links CLI (`bin/docs-link`)**:
  - POSIX shell CLI script operating against `CACHE_BASE` (`${DOCS_CACHE:-$HOME/github/docs-cache/docs/cache}`) and `CACHE_INDEX`.
  - Parses markdown tables in `.index.md` to resolve documentation targets (exact or unique partial matches).
  - Creates relative symlinks inside client projects under `docs/external/<name>` and writes project state to `.docs-links.json` (`cache_path`, `links` key-value pairs, `updated` UTC timestamp).
  - Automatically appends `## External Documentation` usage instructions to client `CLAUDE.md` files upon adding links.
  - Commands: `available` (lists cache catalog with link status), `list` (validates existing symlinks as `OK` or `BROKEN`), `add <name>...` (links docs and updates manifest/CLAUDE.md), `remove <name>` (unlinks and cleans manifest), `sync` (re-creates symlinks from manifest).
- **Library Catalog & Service Metadata**:
  - `library-catalog.md`: Generated documentation catalog referencing 514 libraries (200 JS/TS, 216 Python, 98 Go) mapped to official URL sources.
  - `homelab.yaml`: Schema `homelab.project/v1` project contract defining service metadata (`id: docs-cache`), development lifecycle, `homelab` ownership, `standby` monitoring state, and observation sink (`homelab-api:/observations`).
- **Research & Compliance Artifacts (`docs/findings/`, `docs/research/`)**:
  - Contains repository compliance scan outputs (`repo-compliance-scan-2026-02-07.md`, `scan-results.json`) and caching research documentation (`docs/research/llm-doc-caching/research.md`).

## Canonical entry points
- `bin/docs-link`: Command-line interface binary for managing documentation symlinks (`available`, `list`, `add`, `remove`, `sync`).
- `docs/cache/.index.md`: Central registry index of all cached external documentation.
- `library-catalog.md`: Comprehensive reference mapping 514 external libraries to official documentation URLs.
- `README.md`: Operational overview, integration methods, and quickstart commands.
- `CLAUDE.md`: Mandated agent search order, cache workflow instructions, and webReader integration rules.
- `homelab.yaml`: Homelab project contract specification.
