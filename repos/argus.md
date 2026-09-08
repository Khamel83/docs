# LLM-OVERVIEW — argus
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
Argus is search and web-retrieval infrastructure for AI agents. It provides topology-aware search routing across 14 search provider engines, computed answers via WolframAlpha, adaptive domain memory, universal provenance tracking (`egress`, `machine`, `source_type`), and a 12-step content extraction chain with quality gates. Multi-turn search sessions support conversational context refinement persisted across requests via PostgreSQL or SQLite repositories.

## Machine & Host Ownership
- **Live Status Probe:** Disabled at overview generation time (`JANITOR_RUN_STATUS_PROBE=1` required to execute `scripts/status.py`).
- **Host & Node Topology (`AGENTS.md`):**
  - **Tier 1 (Standalone / Serverless):** Runs on any host with Python 3.11+ (laptop, Mac Mini, Raspberry Pi, cloud VM) using direct API keys and default SQLite persistence (`sqlite:///.../argus.db`).
  - **Tier 2 (Dedicated Hardware Deployment):**
    - **Raspberry Pi 4 (4GB):** Hosts SearXNG aggregator, all local provider adapters, Crawl4AI local JS extraction, and Obscura stealth browser.
    - **Home Server (homelab):** Runs HTTP authority (`ARGUS_NODE_ROLE=primary`) configured with `ARGUS_EGRESS_TYPE=residential` to optimize routing and skip external worker hops.
  - **Node Role Architecture:** Production authority host (`primary`) owns FastAPI services, PostgreSQL storage, provider API keys, and outbox state. Caller nodes (`caller`) connect remotely via stateless CLI or MCP adapters over HTTP/Tailscale TLS with scoped bearer tokens.

## What is actually built
- **Search Provider Adapters & Tier Routing (`argus/providers/`, `argus/broker/`):**
  - **Tier 0 (Free / Unlimited):** DuckDuckGo (public transport policy), SearXNG (aggregates 70+ engines, disabled by default), Yahoo (scraped), GitHub, WolframAlpha (2,000 free computed answer queries/month).
  - **Tier 1 (Monthly Recurring):** Brave (2k/mo), Tavily (1k/mo), Exa (1k/mo), Linkup (1k/mo), Parallel AI (up to 5k/mo for eligible accounts with card on file).
  - **Tier 3 (One-Time Credits):** Serper (2.5k credits), You.com ($20 credit), SearchAPI (unconfigured / missing key), Valyu (account-blocked / disabled).
  - **Routing Policy:** Tier-sorted selection (Tier 0 → Tier 1 → Tier 3) with budget tracking and automatic skipping of exhausted providers.
- **Topology-Aware Egress & Adaptive Domain Memory (`argus/broker/`):**
  - Egress classification (`residential` vs `datacenter`), domain failure memory (auto-routing domain failures from datacenter IPs to residential egress), and universal provenance metadata injection (`egress`, `machine`, `source_type`) into all search and extraction results.
- **12-Step Content Extraction Pipeline (`argus/extraction/`):**
  - Sequential chain: Trafilatura → Crawl4AI → Obscura → Playwright → Jina Reader → Valyu Contents → Firecrawl → You.com Contents → Wayback Machine → archive.is.
  - Quality gates, word count verification, and completeness evaluation between steps. External browser extraction remains fail-closed pending external browser-network attestation.
- **Search Modes (`argus/broker/`):**
  - `discovery`: Canonical sources and related pages.
  - `research`: Broad exploratory retrieval.
  - `recovery`: Dead or moved URL retrieval.
  - `grounding`: Fact-checking and computed-answer generation.
- **Production HTTP Authority API (`argus/api/`, `argus/persistence/`):**
  - FastAPI server delivering `/api/search`, `/api/extract`, `/api/health`, and `/api/budgets`.
  - Authentication via `ARGUS_API_KEY` (caller) and `ARGUS_ADMIN_API_KEY` (admin).
  - PostgreSQL database schema (`0011_extraction_spend_scope`) and Maya outbox dispatch engine.
- **Stateless MCP Server Adapter (`argus/mcp/`):**
  - Stateless HTTP wrapper exposing tools `search_web`, `extract_content`, `recover_url`, and `expand_links`. Does not store credentials or database configuration.
- **Recent Maintained Fixes (v1.6.4 / Sept 2026 commit log):**
  - Routed DuckDuckGo through public transport policy, bound quota and provider probe spend attempts to baked release identity, repaired native HTTP POST framing, normalized UTC Maya outbox timestamps, and fixed compressed provider response decoding.

## Canonical entry points
- **CLI Commands (`argus` CLI via `argus/cli/`):**
  - `argus serve`: Start FastAPI HTTP authority service on `:8000`.
  - `argus mcp serve`: Run stateless MCP adapter (requires `ARGUS_AUTHORITY_URL` and `ARGUS_AUTHORITY_TOKEN`).
  - `argus search -q "<query>" [--mode discovery|research|recovery|grounding] [--session <id>]`: Execute search query.
  - `argus extract -u "<url>"`: Run extraction pipeline on URL.
  - `argus doctor`: Run complete setup, config, provider, connectivity, and MCP diagnostics.
  - `argus health` / `argus budgets` / `argus mcp check`: Inspect provider health, account spend/quotas, and MCP integration.
- **HTTP API Endpoints (`argus/api/`):**
  - `POST /api/search`: Execute search query across provider adapters.
  - `POST /api/extract`: Execute 12-step extraction pipeline.
  - `GET /api/health`: Provider status and system liveness.
  - `GET /api/budgets`: Account spend and quota tracking.
  - Authentication: `Authorization: Bearer $ARGUS_API_KEY` (caller) or `$ARGUS_ADMIN_API_KEY` (admin).
- **MCP Server Tools (`argus/mcp/`):**
  - `search_web`, `extract_content`, `recover_url`, `expand_links`.
- **Python Package (`argus-search`):**
  - Core library structure in `argus/`: `broker`, `providers`, `extraction`, `api`, `cli`, `mcp`, `persistence`.
- **Service Deployment Specs:**
  - `argus.service`: systemd daemon service definition.
  - `docker-compose.yml` / `Dockerfile`: Docker deployment for primary authority host and SearXNG container.
