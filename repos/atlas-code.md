# LLM-OVERVIEW — atlas-code
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
`atlas-code` is a Python-based terminal AI pair programming application (packaged as `aider-chat` / Aider). It enables interactive LLM-driven coding in terminal environments, supporting model integration (OpenAI, Anthropic, OpenRouter, Grok-4, Gemini, DeepSeek, Kimi K2), git-aware automated diff patching, polyglot benchmarking, and repository-wide context management.

## Machine & Host Ownership
Project metadata is registered under `homelab.yaml` (`schema: homelab.project/v1`, project `atlas-code`, lifecycle `development`, monitoring `standby`). Runtime production host is specified as `unknown` with unit `atlas-code`. No multi-host deployment information is available from the status probe.

## What is actually built
- **Core Engine & CLI (`aider/`)**:
  - `aider.main:main`: Main interactive loop and CLI entry point.
  - `aider.args` & `aider.args_formatter`: Command-line options parsing and help formatting.
  - `aider.coders`: Code editing engine supporting diff-based edits, whole-file replacements, and function-level AST updates.
  - `aider.commands`: Interactive in-chat command processor (`/commit`, `/diff`, `/help`, etc.).
  - `aider.diffs` & `aider.copypaste`: Git patch creation, diff formatting, and clipboard utilities.
  - `aider.analytics`: Usage tracking and telemetry gathering.
- **Polyglot Benchmark Engine (`benchmark/`)**:
  - `benchmark/benchmark.py`: Typer CLI application for executing multi-language coding benchmark tasks.
  - `benchmark/over_time.py`: Performance visualization utility tracking model accuracy trends (`ModelData` class).
  - `benchmark/problem_stats.py`: Benchmark result aggregator evaluating leaderboard outputs (`polyglot_leaderboard.yml`).
  - `benchmark/refactor_tools.py`: Python AST transformation and verification suite using `ast.NodeTransformer`.
  - `benchmark/swe_bench.py`: SWE-bench metric extractor and chart generator.
  - `benchmark/rungrid.py`: Matrix runner for evaluating model and edit format combinations.
  - Test scripts & isolation: `clone-exercism.sh`, `cpp-test.sh`, `npm-test.sh`, `docker.sh`, and `benchmark/Dockerfile`.
- **Scripts & Site Generators (`scripts/`)**:
  - Site & README tooling: `scripts/homepage.py` (dynamic badge output via Cog), `scripts/history_prompts.py`, `scripts/clean_metadata.py`, `scripts/dl_icons.py`.
  - Jekyll documentation builds: `scripts/jekyll_build.sh` and `scripts/Dockerfile.jekyll`.
- **Packaging & Project Contract**:
  - `pyproject.toml`: Configures package `aider-chat` targeting Python `>=3.10,<3.13` with `setuptools_scm`. Optional dependency extras include `dev`, `help`, `browser`, and `playwright`.
  - `homelab.yaml`: Homelab project contract specifying project ID `atlas-code`, health source `homelab-api`, and secret consumer configuration.

## Canonical entry points
- `aider`: Primary executable script mapped to `aider.main:main` (via `pyproject.toml`).
- `python3 -m aider`: Direct package execution via `aider/__main__.py`.
- `python3 benchmark/benchmark.py`: CLI harness for running polyglot benchmark suites.
- `python3 -m pytest`: Suite runner configured via `pytest.ini` covering unit and integration tests under `tests/`.
- `homelab.yaml`: Infrastructure definition contract for homelab project management.
