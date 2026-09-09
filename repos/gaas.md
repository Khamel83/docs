# LLM-OVERVIEW — gaas
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
GAAS (Gami As A Service) is an automated sports analytics platform designed to calculate rarity scores and identify statistically rare individual game performances across multiple professional sports leagues (NFL, NBA, MLB, F1, NHL, and UEFA Champions League). It ingests raw box scores and play-by-play datasets (such as nflfastR), computes historical probability distributions and rarity metrics across positions and statistical buckets, and outputs static JSON feeds and an HTML web dashboard published to `gaas.zoheri.com` (GitHub Pages).

## Machine & Host Ownership
According to `homelab.yaml`, GAAS is configured to run on host `homelab` as systemd unit `gaas` with `homelab-native` sync mode and standby monitoring status. No multi-host deployment or live status probe information is active.

## What is actually built
The repository consists of data collectors, statistical rarity processors, dataset generators, static HTML presentation layers, and deployment utilities:

- **Core Python Multi-Sport System (`src/`)**:
  - `src/gaas_unified.py`: Primary CLI orchestrator that executes data collection and rarity processing across all sports.
  - `src/gaas_multi_position.py`: Multi-position analytics runner focusing on NFL positional breakdowns (QB, RB, WR, TE).
  - Sport-specific runners: `src/gaas_nba.py`, `src/gaas_mlb.py`, `src/gaas_f1.py`, `src/gaas_nhl.py`, `src/gaas_champions_league.py`, `src/gaas.py`.
  - Submodules in `src/collectors/` (e.g., `NFLCollector` for fetching game stats) and `src/generators/` / processors (e.g., `processors.nfl_rarity.NFLRarityEngine`).

- **Data Processing & Harvesting (`scripts/`)**:
  - `scripts/download_complete_nfl.py`: Downloads historical nflfastR play-by-play data (1999–2024).
  - `scripts/download_all_nfl_positions.py`, `scripts/download_game_stats.py`: Downloads positional statistics.
  - `scripts/load_all_positions_archive.py`: Creates positional SQLite tables (`qb_games`, `rb_games`, `wr_games`, `te_games`) in `data/archive/nfl_archive.db`.
  - `scripts/data_processor.py`, `scripts/corrected_data_processor.py`: Aggregates SQLite position databases, calculates bucket probabilities, and computes statistical rarity indices.
  - `scripts/create_sample_nba_data.py`, `scripts/create_sample_mlb_data.py`, `scripts/create_sample_f1_data.py`: Generates sample/fallback datasets for non-NFL sports.

- **Storage & Results Artifacts**:
  - SQLite database target: `data/archive/nfl_archive.db`.
  - Per-sport JSON outputs: `nfl/` (`qb_all_time.json`, `qb_latest.json`, `rb_all_time.json`, etc.), `nba/`, `mlb/`, `f1/`, `nhl/`, `champions_league/`.
  - Analysis syntheses & feeds in `results/`: `honest_overview.json`, `enhanced_index.json`, `enhanced_nfl_analysis.json`, `honest_final_interface.json`.

- **Web Dashboard & Static Site**:
  - Root web components: `index.html`, `test_nba.html`, `.nojekyll` (configured for GitHub Pages deployment).
  - `docs/`: Secondary web site output including `docs/index.html` and static data assets (`docs/index.json`, `docs/nfl`, `docs/nba`, `docs/mlb`, `docs/f1`, `docs/champions_league`).

- **Systemd & Deployment**:
  - `scripts/deploy_systemd.sh`: Installs systemd service files for automated background harvesting.
  - `homelab.yaml`: Project contract defined under schema `homelab.project/v1`.

- **Testing Infrastructure (`tests/`)**:
  - `tests/test_integration.py`: Validates end-to-end execution of `src/gaas_unified.py` and individual sport drivers.
  - `tests/test_all_sports.py`: Exercises `NFLCollector` and `NFLRarityEngine` logic.

## Canonical entry points
- **Unified Pipeline Orchestrator**: `python src/gaas_unified.py`
- **Positional & Sport Runners**: `python src/gaas_multi_position.py`, `python src/gaas_nba.py`, `python src/gaas_mlb.py`, `python src/gaas_f1.py`
- **Data Harvesting & SQLite Ingestion**: `python scripts/download_complete_nfl.py`, `python scripts/load_all_positions_archive.py`
- **Rarity Processing & Analysis**: `python scripts/corrected_data_processor.py`, `python scripts/data_processor.py`
- **Test Suite**: `pytest tests/`
- **Systemd Deployment**: `bash scripts/deploy_systemd.sh`
