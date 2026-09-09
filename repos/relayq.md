# LLM-OVERVIEW — relayq
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
GitHub-first job orchestration platform and hybrid runner kit. Uses GitHub Actions' native workflow queueing and polling self-hosted runners (Mac mini, Raspberry Pi) to execute local compute workloads (audio/video transcoding, podcast transcription, batch file processing) without open inbound network ports or dedicated queue brokers.

## Machine & Host Ownership
- **OCI VM (Oracle Cloud Infrastructure)**: Dispatch node; triggers workflows via GitHub API and `gh` CLI commands.
- **Mac mini**: Heavy processing runner node (`osx-x64`), configured via `actions-runner` for CPU/GPU-intensive transcoding and transcription workloads.
- **Raspberry Pi 4 / Raspberry Pi 3**: Light task processing runner nodes (`runner` user) for low-overhead task execution.
- **Ubuntu Host (`/home/ubuntu/dev/atlas/`)**: Host environment housing the Atlas SQLite database (`podcast_processing.db`) queried by `atlas_data_provider.py`.
- **Tailscale Funnel**: Optional zero-cost ingress module establishing private mesh network connectivity to internal resources.

## What is actually built
- **Target Selection & Routing (`bin/select_target.py`, `policy/policy.yaml`)**: Python script that parses `policy/policy.yaml` and evaluates job metadata (such as `max_size_mb`) against constraints to select the appropriate target GitHub Actions workflow file.
- **Job Dispatching (`bin/dispatch.sh`, `jobs/transcribe.sh`)**: CLI entry points for pushing processing jobs to the GitHub Actions job queue via `gh workflow run`. `jobs/transcribe.sh` handles worker execution logic on self-hosted runners.
- **Atlas Podcast Integration (`atlas_data_provider.py`)**: Data provider connecting to SQLite database at `/home/ubuntu/dev/atlas/podcast_processing.db`. Queries `episodes` and `podcasts` tables (`WHERE processing_status = 'pending'`) to supply pending podcast audio jobs and accept completion results back into the database.
- **Tailscale Funnel Infrastructure (`tailscale-funnel-module/`)**: Integrated zero-cost infrastructure stack with setup guides (`QUICKSTART.md`, `INTEGRATION_GUIDE.md`), management scripts, templates, and example configurations.
- **Homelab Project Contract (`homelab.yaml`)**: `homelab.project/v1` specification declaring service `relayq` under `homelab` ownership, standby monitoring state, and `homelab-api:/observations` event sink.
- **Examples & Helper Tools (`examples/`)**: Automation scripts including `transcode_video.py`, `transcribe_audio.py`, `transcode_batch.py`, and `custom_command.py`.
- **Retired Infrastructure (`legacy/`, `setup.py`)**: *Retired History* — Legacy Celery and Redis worker implementation (`legacy/relayq/worker.py`, `legacy/relayq/tasks.py`, `legacy/relayq/config.py`, `setup.py` dependencies `celery[redis]` and `redis`) has been deprecated and moved to `legacy/`. Active architecture relies strictly on GitHub native runner queues.

## Canonical entry points
- `bin/dispatch.sh`: Shell script to dispatch job workflows using `gh`.
- `bin/select_target.py`: Target selection script evaluating job parameters against `policy/policy.yaml`.
- `atlas_data_provider.py`: Python interface script fetching pending podcast episodes and updating status in `/home/ubuntu/dev/atlas/podcast_processing.db`.
- `Makefile`:
  - `make help`: Display available targets.
  - `make dispatch URL=<url>`: Test workflow job dispatch.
  - `make check`: Run system verification checks.
  - `make fmt`: Format shell scripts (`shfmt`) and Python code.
- `jobs/transcribe.sh`: Runner-side shell script executing transcription tasks.
- `tailscale-funnel-module/scripts/`: Setup and management scripts for Tailscale Funnel.
- `~/.config/relayq/env`: Unified environment configuration file for credentials and routing variables.
