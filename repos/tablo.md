# LLM-OVERVIEW — tablo
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
Automated Over-The-Air (OTA) TV recording processing pipeline for Tablo DVR devices. Discovers raw recordings, strips commercials via Comskip, resolves show/episode metadata using cached TVMaze EPG schedules, uses local Whisper transcription and Ollama LLM disambiguation for ambiguous matches, and moves formatted MP4 files into a Plex TV library. Supports legacy HTTP streaming extraction, direct USB drive ingestion (for 4th Gen Tablo firmware 2.2.55+), and a distributed RPi4 drive monitor to Mac Mini processing pipeline.

## Machine & Host Ownership
- **Homelab Registry (`homelab.yaml`)**: Project ID `tablo` (kind: `service`, unit: `tablo`), owner `homelab`. Production host listed as `unknown`.
- **Mac Mini Host**: Acts as the primary media processing engine, running `scripts/run_all_once.sh` or background socket listener `macmini/network_receiver.py` (TCP 2222). Holds binary tools (FFmpeg, Comskip, Whisper, Ollama) and target Plex directory (`/Users/Shared/Plex/TV/`). Provisioned via `macmini/setup_macmini.sh`.
- **Raspberry Pi 4 Host**: Acts as a remote USB ingestion node running `rpi4/auto_tablo_processor.py` via systemd unit `rpi4/usb_monitor.service`. Monitors USB mount points (`/media/*`, `/mnt/*`), copies raw Tablo recordings, and syncs over network sockets to Mac Mini. Provisioned via `rpi4/setup_rpi4.sh`.

## What is actually built
### Core Subsystems (`src/`)
- **`epg_cache.py` (`TVMazeCache`)**: Downloads US schedule data from TVMaze API (`https://api.tvmaze.com/schedule`) for configured major networks (PBS, CBS, NBC, ABC, FOX) across a sliding window (`days_back: 7`, `days_forward: 14`). Caches normalized JSON episodes in `/opt/tablo/epg/`. Matches recordings using start time tolerance (`start_window: 300s`), duration tolerance (`duration_window: 120s`), and minimum score threshold (`min_score: 0.75`).
- **`pull_from_tablo.py` (`TabloPuller`)**: Legacy HTTP network streaming puller for Tablo port 18080 API. Discovers HLS streams (`/pvr/`), downloads playlists with FFmpeg to `/opt/tablo/raw/`, runs Comskip commercial removal to produce clean MP4s in `/opt/tablo/clean/`, and generates initial sidecar metadata in `/opt/tablo/meta/`.
- **`pull_from_tablo_drive.py` (`TabloDrivePuller`)**: Direct USB drive mode for 4th Gen Tablo devices (firmware 2.2.55+ HTTP block). Detects mounted USB drives (default `/Volumes/Tablo`), scans `recordings/<numeric_id>` directories for transport stream segments, calculates duration/timestamps, copies files, executes Comskip, and writes metadata JSON to `/opt/tablo/meta/`. Tracks state in `/opt/tablo/state.json`.
- **`identify_and_rename.py` (`RecordingIdentifier`)**: Loads clean metadata (`status == 'clean'`), queries EPG cache for matching shows. If multiple candidate EPG entries match a recording time window:
  1. Invokes OpenAI Whisper CLI (`whisper <file> --model base --language en --output_format json`) to extract audio transcript.
  2. Queries local Ollama LLM (`llama3:8b` at `http://localhost:11434/api/chat`, up to 3 retries) with transcript excerpt (first 500 chars) and candidate titles to choose the best episode match.
  3. Formats Plex standard filename (`Show Name - S01E02 - Episode Title.mp4`), creates show subdirectory, moves video file to `/Users/Shared/Plex/TV/<Show Name>/`, and updates metadata status to `moved_to_plex`.
- **`tablo_auth.py` (`TabloAuth`)**: Helper module providing web header spoofing, discovery, and HTTP authentication logic for 4th Gen Tablo firmware interfaces.

### Distributed Receiver & Monitor (`macmini/` & `rpi4/`)
- **`rpi4/auto_tablo_processor.py` (`TabloRPi4Processor`)**: Service daemon on RPi4 watching mount points for inserted Tablo storage. Copies raw directories to `/opt/tablo/recordings/`, updates `/opt/tablo/processor_state.json`, and triggers network socket events to the Mac Mini receiver.
- **`rpi4/usb_monitor.service`**: Systemd unit running RPi4 auto-processor continuously.
- **`macmini/network_receiver.py` (`MacMiniReceiver`)**: Multithreaded socket daemon listening on Mac Mini (port 2222). Receives `new_recording` payload events from RPi4, moves files into `/opt/tablo/incoming/`, and triggers identification and EPG matching routines.

### Data Structures & Configuration
- **`config.yaml`**: Configures operational mode (`tablo.mode`: `direct_drive` | `streaming` | `network`), file paths (`raw_dir`, `clean_dir`, `plex_tv_root`, `meta_dir`, `epg_dir`, `logs_dir`, `state_file`), binary tool names (`ffmpeg`, `ffprobe`, `comskip`, `whisper`, `ollama`), LLM settings, EPG network filters, and match tolerance windows.
- **Sidecar Metadata (`/opt/tablo/meta/<recording_id>.json`)**: Contains fields `id`, `start_time_utc`, `end_time_utc`, `duration_seconds`, `clean_file`, `status`, `epg_match`, `final_path`, and `moved_at`.
- **State File (`/opt/tablo/state.json`)**: Contains JSON object `{ "processed_ids": [...], "last_epg_fetch": "<ISO8601>" }`.

## Canonical entry points
- `./scripts/run_all_once.sh [config_file]`: Primary orchestration script. Executes `epg_cache.py`, triggers the appropriate puller (`pull_from_tablo_drive.py`, `pull_from_tablo.py`, or network incoming directory check based on `tablo.mode`), and runs `identify_and_rename.py`.
- `python3 src/identify_and_rename.py [config_file]`: Runs standalone EPG identification, Whisper transcription fallback, Ollama LLM candidate selection, and Plex library file moves.
- `python3 src/pull_from_tablo_drive.py [config_file]`: Executes direct USB drive extraction from connected Tablo drive at configured mount point.
- `python3 src/pull_from_tablo.py [config_file]`: Executes HTTP network streaming pull from legacy Tablo device.
- `python3 src/epg_cache.py [config_file]`: Downloads and refreshes TVMaze schedule cache for configured networks into EPG directory.
- `python3 macmini/network_receiver.py`: Runs background socket daemon on Mac Mini receiving sync notifications from RPi4.
- `python3 rpi4/auto_tablo_processor.py`: Runs continuous USB drive monitoring and sync engine on RPi4 host.
- `python3 test_tablo.py [ip]`: Diagnostic script testing HTTP endpoints and user-agent responses on a Tablo device IP.
- `./install.sh`: Setup script creating `/opt/tablo/` directory hierarchy and installing Python dependencies (`requirements.txt`).
- `macmini/setup_macmini.sh` & `rpi4/setup_rpi4.sh`: Provisioning scripts setting up host-specific paths, packages, and system daemon services.
