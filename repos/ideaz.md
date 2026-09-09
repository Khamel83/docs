# LLM-OVERVIEW — ideaz
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
- **Personal Portfolio Intelligence & Idea Workbook**: Automated system that maps GitHub user repositories (`Khamel83`) and starred repositories into structured, multi-layered AI agent context files and reference cards.
- **Primary Operational Function**: Ingests GitHub API metadata, scans local repository clones (`/home/ubuntu/github`), auto-classifies repositories by theme/category, executes LLM summarization via OpenRouter free tier models, and builds machine-readable context artifacts (`llms.txt`, `llms-full.txt`, `stars-context.txt`, `workbook.md`, `daily-diff.md`, `mega-context.md`).
- **Core Design Invariants**: Maintains strict separation between authored repositories ("My Projects") and starred repositories ("Starred Projects"). Preserves user manual state overrides (`category_override`, `notes`, `why_starred`, `infra`) across API sync cycles.

## Machine & Host Ownership
- No multi-host deployment topology or multi-machine cluster is declared in code or configuration.
- `config.yaml` sets `local_repos_dir: /home/ubuntu/github`, indicating local execution on an Ubuntu workstation or server instance.
- `homelab.yaml` defines `homelab.project/v1` metadata with production host as `unknown`, managing service unit `ideaz` under `standby` monitoring state. Status probe is disabled by default (`JANITOR_RUN_STATUS_PROBE=1` required for status scripts).

## What is actually built
- **Core Processing Engine (`ideaz.py`)**:
  - Python 3 single-file CLI application using `click`, `requests`, `pyyaml`, `subprocess`, `hashlib`, `json`, and `pathlib`.
  - Manages GitHub API ingestion, local repository file scanning, SHA-256 hash tracking, OpenRouter LLM interactions, Markdown report generation, and automated git workflow.
- **Data Persistence & Manual Overrides (`state.json`)**:
  - Primary uncommitted JSON state storage with keys:
    - `repos`: Dict of authored repositories (`name`, `description`, `html_url`, `language`, `stargazers_count`, `updated_at`, `pushed_at`, `category`, `category_override`, `notes`, `infra`, `llm_summary`, `llm_architecture`, `llm_tech_stack`, `llm_key_features`).
    - `stars`: Dict of starred repositories (`name`, `owner`, `full_name`, `description`, `html_url`, `language`, `stargazers_count`, `category`, `notes`, `why_starred`, `llm_summary`, `llm_key_features`, `llm_tech_stack`, `llm_how_it_works`, `llm_why_starred`, `llm_integration_ideas`).
    - `ideas`: Array of captured idea records (`id`, `text`, `timestamp`, `archived`, `related_repos`).
    - `file_hashes`: Dictionary mapping repo -> relative file path -> SHA-256 digest for incremental change detection.
    - `scans`: Cache of local repo scan results (agent configs, build manifests, shell scripts, commit counts).
- **Classification Engine (`classify_entry`, `classify_score`)**:
  - Categorizes repositories via keyword scoring into seven categories (`DEFAULT_CATEGORIES`): `ai-agents`, `infra-devops`, `devtools`, `apps-web`, `data-analytics`, `security-privacy`, and `misc`.
- **Local Repo & File Hash Scanner (`scan_repo_context`, `enrich_from_local`, `compute_file_diff`)**:
  - Scans local repository clones in `LOCAL_REPOS_DIR` (`/home/ubuntu/github`), filtering noise directories (`.git`, `node_modules`, `.venv`, `.janitor`, `.claude`).
  - Scans for agent instructions (`CLAUDE.md`, `AGENTS.md`, `.cursorrules`), build manifests (`package.json`, `pyproject.toml`, `requirements.txt`), infra files (`Makefile`, `docker-compose.yml`), and shell scripts (`*.sh`).
  - Parses TOML dependencies from `pyproject.toml` via regex (`extract_toml_dependencies`).
  - Computes 90-day git commit activity counts (`count_recent_commits`).
  - Tracks SHA-256 file hashes (`compute_file_hash`); `needs_resummary` flags repositories for LLM re-summarization when tracked build/agent files change.
- **LLM Integration & Fallback Subsystem (`llm_call`, OpenRouter API)**:
  - Integrates with OpenRouter completions endpoint (`https://openrouter.ai/api/v1/chat/completions`) using free tier model arrays with deterministic fallback (e.g. `google/gemini-2.0-flash-lite-001`, `meta-llama/llama-3.3-70b-instruct:free`, `qwen/qwen-2.5-coder-32b-instruct:free`).
  - `llm_summarize_repo`: Produces structured summary, tech stack, architecture, and feature bullets for authored repos.
  - `llm_map_relationships`: Evaluates cross-project relationships across all user repos in a single LLM pass.
  - `llm_summarize_star`: Generates star reference cards with personalized "Why I Starred This" and "Integration Ideas" contextualized against `build_user_repos_context`.
  - `sanitize_content`: Regex scrubbing of API keys, tokens, and secret patterns (`SECRET_PATTERNS`) prior to artifact generation.
- **Multi-Tier Context Artifact Generators**:
  - `output/workbook.md` (`generate_workbook`): Main living portfolio catalog split into authored projects and starred repos.
  - `output/llms.txt` (`generate_llms_txt`): Compact index (~1K tokens) adhering to `llmstxt.org` specification.
  - `output/llms-full.txt` (`generate_llms_full_txt`): Expanded context for authored repos (~8K tokens) containing file trees and agent instructions.
  - `output/stars-context.txt` (`generate_stars_context_txt`): Index of starred repositories with personalized LLM context.
  - `output/repos/{name}.md` (`generate_all_repo_cards`): Per-repo reference cards featuring scanner data, shell scripts, dependencies, and portfolio relationship links.
  - `output/stars/{name}.md` (`generate_all_star_cards`): Per-star reference cards with personalized LLM analysis.
  - `output/daily-diff.md` (`generate_daily_diff`): Daily change ledger tracking API changes and modified local files.
  - `output/mega-context.md` (`dump`, `repomix_to_string`): Single-file mega context bundling full `repomix` dumps for top 8 active local repos (8MB cap per repo) and brief blocks for remaining repos (excluding data/infra repos `divorce`, `ipeds`, `atlas`, `homelab`).
- **Idea Capture Subsystem (`add_idea`, `list_ideas`, `archive_idea`)**:
  - Stores text ideas in `state.json`, automatically linking relevant user repos by matching idea text against repo names and category keywords.
- **Automated Watcher & Cron Entry Point (`watch`)**:
  - Combined workflow: executes fetch, context generation, daily diff computation, outputs JSON summary to stderr, and triggers `git commit` and `git push` if `auto_push: true`.
- **Auxiliary Scripts & Docs (`scripts/`, `docs/`)**:
  - `scripts/secrets-helper.sh`: SOPS/Age secrets management helper.
  - `scripts/oneshot-check.sh`: System status check script.
  - `scripts/skillsmp-search.sh`: Skill search utility script.
  - `docs/sessions/*.md.age`: SOPS/Age encrypted interactive session history files.
- **Dependencies**:
  - Python runtime packages: `click`, `requests`, `pyyaml`.
  - System CLI tools: `git` (activity checks, auto-push), `repomix` (optional, for mega-context generation).

## Canonical entry points
- **CLI Commands (`python3 ideaz.py <command>`)**:
  - `python3 ideaz.py` / `python3 ideaz.py fetch`: Pull user repos and starred repos from GitHub API (`api.github.com`), merge with `state.json`, print diff.
  - `python3 ideaz.py generate`: Rebuild `output/workbook.md` from cached state.
  - `python3 ideaz.py context [--llm | --no-llm]`: Regenerate context layers (`llms.txt`, `llms-full.txt`, `stars-context.txt`, `output/repos/*.md`, `output/stars/*.md`).
  - `python3 ideaz.py watch`: Automated entry point (cron) — executes fetch, context update, diff generation, stderr JSON output, and git auto-push.
  - `python3 ideaz.py diff`: Display API and local file changes since last execution.
  - `python3 ideaz.py idea ["<text>" | --list | --archive <id>]`: Record, list, or archive personal ideas.
  - `python3 ideaz.py dump [--hot 8] [--days 90] [--force]`: Execute `repomix` on active repos to generate `output/mega-context.md`.
- **Config & State Files**:
  - `config.yaml`: Runtime settings (`user`, `token`, `local_repos_dir`, `auto_push`).
  - `homelab.yaml`: Project metadata contract (`schema: homelab.project/v1`).
  - `state.json`: Primary data cache (repos, stars, ideas, file hashes).
  - `output/`: Folder containing all generated markdown context artifacts.
- **Orchestration & Agent Governance**:
  - `AGENTS.md`: ONE_SHOT v14 orchestration control plane constitution defining operators (`/short`, `/full`, `/conduct`), dispatch protocol, task classes, intelligence tiers (`glm_claude`, `codex`, `gemini_cli`, `free`), and CLI entry points (`shot`, `zai`, `or`).
