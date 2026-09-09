# LLM-OVERVIEW — gamez
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
`gamez` is a gaming library management, ROM inventory tracking, emulation sync, and retro handheld configuration automation repository. It manages a master game catalog (`game_library_master.xlsx`), indexes local ROM storage, performs automated title normalization and fuzzy matching, searches Prowlarr for missing ROMs, syncs curated libraries across machines, and automates 2-card firmware and ROM deployment for Anbernic retro handheld devices (specifically the RG40XXH). It also houses `oneshot` (OneShot v14), a multi-model task classification and parallel dispatch control plane.

## Machine & Host Ownership
- **`homelab` (`homelab-ts`)**: Linux server acting as primary ROM storage (`/mnt/main-drive/roms`, `/mnt/main2/ROM/anbernic-device/NO NAME/Roms`), host for master SQLite file index (`master-index.db`), host for Prowlarr indexer search API (`http://localhost:9696/prowlarr`), and execution environment for python inventory generator scripts (`/home/khamel83/github/gamez/`).
- **`macmini`**: macOS workstation hosting local repository clone (`/Users/macmini/github/gamez`), running SD card formatting/creation scripts using macOS `diskutil`, hosting Dolphin and OpenEmu emulation setups (`~/Games/ROMs`), and serving SteamLink launchers (`~/Games/Launchers`).
- **Anbernic RG40XXH**: Target portable handheld console running a 2-card setup (TF1 system card running cbepx-me Stock Mod v3.9.8 64-bit firmware with simple list menu mode; TF2 games card formatted FAT32 running MinUI base firmware v20251127-1 and curated ROM folders).

## What is actually built
- **Core Inventory Engine (`gamez/inventory.py`)**: Parses master games list from `game_library_master.xlsx`, scans ROM files from SQLite index (`master-index.db`) or direct filesystem paths, normalizes titles, and computes string token similarity matches against catalog games (`match_game_to_rom`). Outputs structured summary reports to `reports/full-inventory.json`, `reports/by-system.md`, and `reports/launch-checklist.md`.
- **Prowlarr Missing ROM Searcher (`scripts/search-roms.py`)**: Reads games marked `[NEED]` in `reports/by-system.md`, maps retro systems to optimized torrent search terms, queries Prowlarr API (`/prowlarr/api/v1/search`), and writes output to `reports/search-results.json` and `reports/search-results.md`.
- **Gamez CLI Wrapper (`scripts/gamez`)**: Bash front-end exposing operational tasks (`status`, `scan`, `search`, `sync`, `setup-mac`, `need`, `have`). Reads `full-inventory.json` directly to display library completion metrics and phase readiness.
- **Mac Emulation Sync & Config (`scripts/sync-to-mac.sh`, `scripts/setup-mac-emulators.sh`)**: Syncs ROM files from homelab storage to Mac Mini over SSH (`rsync`), configures Dolphin `Dolphin.ini` search paths, creates OpenEmu folder trees, and generates executable Steam launchers (`LaunchDolphin.sh`, `LaunchOpenEmu.sh`) for Apple TV streaming.
- **RG40XXH Setup & Tactical Fix (`oneshot/setup-rg40xxh.sh`, `oneshot/fix-roms.sh`)**: Automated macOS script that detects SD cards, formats FAT32/MBR via `diskutil`, fetches MinUI base archives from GitHub releases, maps system names (`FC`, `SFC`, `MD`, `GBA`, `GB`, `GBC`, `PS`), and transfers verified Phase 1 ROMs via SSH from homelab. Includes a tactical fix script to patch ROMs on mounted cards without re-flashing firmware.
- **OneShot v14 Control Plane (`oneshot/`)**: Multi-model agent orchestration framework containing task category classification (`oneshot/core/task_schema.py`), machine-readable planning structures (`oneshot/core/plan_schema.py`), parallel model runner (`oneshot/core/dispatch/run.py`), and background session event recorder (`oneshot/core/janitor/`).

## Canonical entry points
- **Gamez CLI**: `scripts/gamez <status|scan|search|sync|setup-mac|need|have>`
- **Inventory Scanner**: `python3 gamez/inventory.py`
- **Prowlarr Search Script**: `python3 scripts/search-roms.py`
- **Emulation Sync**: `scripts/sync-to-mac.sh [--dry-run] [system]`
- **Emulator Configurator**: `scripts/setup-mac-emulators.sh`
- **RG40XXH Automated Setup**: `oneshot/setup-rg40xxh.sh [--test|--match]`
- **RG40XXH Tactical ROM Fix**: `oneshot/fix-roms.sh`
- **OneShot Parallel Runner**: `python3 -m core.dispatch.run` (from `oneshot/`)
- **Master Excel Catalog**: `game_library_master.xlsx`
- **Project Metadata**: `homelab.yaml`
- **Handheld Handoff Documentation**: `HANDOFF_RG40XXH.md`
