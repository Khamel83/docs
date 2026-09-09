# LLM-OVERVIEW — dada
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
Dada is a private family communication web application designed to allow kids (e.g., Ramy & Alex) to send text messages, audio recordings, video files (with real-time filters), and arbitrary file uploads to a parent via Telegram. It supports WebRTC peer-to-peer video calling, presence tracking, Google Meet integration, local disk and Dropbox file backups, and California-only IP geolocation filtering with file extension security guards. The architecture supports high-availability dual deployment: a full-featured FastAPI server running on a Linux host (OCI) paired with a serverless fallback on Vercel, coordinated by a Cloudflare Worker edge failover script.

## Machine & Host Ownership
Status probe is disabled (`JANITOR_RUN_STATUS_PROBE=1`) and `AGENTS.md` is absent. `homelab.yaml` records `runtime.production.host: unknown` and `managed_by: unknown` for systemd unit `dada`. Service configuration (`dada.service`, `dada-cleanup.service`) specifies execution under user `ubuntu` at `/home/ubuntu/dev/dada` or `/home/ubuntu/github/dada`. Edge configuration (`cloudflare-worker.js`) routes traffic to an Oracle Cloud Infrastructure (OCI) origin (`https://dada-direct.khamel.com`) with automatic failover to Vercel (`https://dada-khamel-fallback.vercel.app`).

## What is actually built

### 1. Core Server & Web API (`app.py`)
- **Framework & Runtime**: FastAPI application running on Uvicorn (bound to `0.0.0.0:8000`). Mounted static files from `static/` and local uploads from `/home/ubuntu/uploads`. Jinja2 templates served from `templates/`.
- **UI & Multi-Kid Support**: Main page (`GET /`) loads `templates/index.html` with client-side theme selection (`ninjago`, `lego`, `mario`, `pokemon`, `minnie`). Kid names and color palettes parsed from environment variable `KIDS`.
- **Media Upload Pipeline**:
  - `POST /upload/video`: Accepts video uploads, invokes `ffmpeg` to convert `.webm` files to `.mp4` (`convert_webm_to_mp4`), sends Telegram notification with media payload, and backs up files locally or to Dropbox (`dbx` via `DROPBOX_ACCESS_TOKEN`).
  - `POST /upload/audio`: Handles voice message uploads and Telegram relay.
  - `POST /upload/text`: Relays text messages directly to Telegram chat.
  - `POST /upload/file`: Accepts generic file uploads after passing security validation.
- **Real-Time Communication & Presence**:
  - `GET /chat`, `POST /telegram/send`, `POST /telegram/webhook`: Real-time Telegram messaging interface. Webhook endpoint processes parent replies from Telegram and broadcasts them to connected WebSocket clients.
  - `WS /ws/chat/{kid}`: Managed by `SimpleChatManager` (`chat_manager.py`). Holds up to 10 offline messages for 1 hour and broadcasts incoming parent replies to active kid connections.
  - `GET /video`, `WS /ws/video/{room_id}`: WebRTC video calling page (`templates/video_call.html`) and signaling WebSocket endpoint. Enforces a 5-minute (300s) notification debounce for video calls to prevent spamming Telegram.
  - `WS /ws/presence/{kid_name}`: Tracks active kid client connections (`presence_connections`) and timestamps (`kid_last_active`) for presence-aware communication.
  - `POST /start-meet`: Generates Google Meet links and notifies via Telegram.
  - `GET /dad-audio/{audio_uuid}`: Serves temporary audio recordings created by dad.
- **Admin & Utility Endpoints**:
  - `GET /health`: System health status returning disk usage, uptime, service flag statuses, and timestamp.
  - `GET /simple`: No-JavaScript HTML upload form fallback.
  - `GET /files`: Interactive HTML dashboard for downloading and managing uploaded files.

### 2. Security & Access Control (`security.py`)
- **IP Geolocation Filtering**: `is_california_ip(request)` queries `ip-api.com` or `ipapi.co` with an LRU cache (`get_cached_ip_key`). Restricts file uploads strictly to California (`CA`) IP addresses with full US region fallback if region detection is ambiguous.
- **Extension Validation**: `is_file_type_allowed(filename)` enforces an explicit extension blocklist (`BLOCKED_EXTENSIONS`: `.exe`, `.sh`, `.py`, `.js`, `.zip`, `.env`, etc.) and permits safe file formats (`ALLOWED_EXTENSIONS`: media, images, documents).
- **Enforcement Handler**: `validate_upload(request, filename)` integrates IP and file checks into upload routes.

### 3. Encryption Subsystem (`encryption.py`)
- **Session Logging**: `EncryptedLogger` uses AES-256 (`Fernet`) encryption. Key derived via `PBKDF2HMAC` (SHA-256) using a configurable secret salt (`LOG_ENCRYPTION_SALT`).

### 4. Serverless & Failover Architecture
- **Vercel Adapter (`api/index.py`, `api/vercel_app.py`, `vercel.json`)**: Serverless FastAPI wrapper (`api/vercel_app.py`) running Python 3.11 runtime (`@vercel/python`). Handles text, audio, and file uploads directly to Telegram when the main server is offline. Uses local `/tmp/dada_uploads` for transient storage.
- **Cloudflare Edge Failover (`cloudflare-worker.js`, `cloudflare-worker-complete.js`)**: Edge worker intercepting inbound requests to `dada.khamel.com`. Performs health checks against `OCI_ORIGIN` (`https://dada-direct.khamel.com/health`) with a 15-second cache and 5000ms timeout. Automatically fails over traffic to `VERCEL_ORIGIN` (`https://dada-khamel-fallback.vercel.app`) or serves embedded fallback HTML (`FALLBACK_HTML`) when OCI is down.

### 5. Automated Operations & Maintenance
- **File Retention (`auto_cleanup.py`)**: `DadaCleanup` monitors disk space via `shutil.disk_usage`. Automatically deletes files older than 30 days or executes emergency purge when root filesystem disk usage exceeds 80%.
- **Deployment Validation (`validate_deployment.py`)**: `DeploymentValidator` verifies environment configuration, directory structure, system dependencies (`ffmpeg`), and endpoint responsiveness.
- **System Services**:
  - `dada.service`: Systemd unit executing `app.py` inside virtualenv (`/home/ubuntu/dev/dada/venv`).
  - `dada-cleanup.service` & `dada-cleanup.timer`: Executes `auto_cleanup.py` on schedule.
  - `dada-monitor.service`: System auto-healer running `monitor_and_heal.py`.
  - `Caddyfile`: Reverse proxy config mapping `dada.khamel.com` to `localhost:8000`, setting a 100MB body limit, gzip compression, HSTS, and security headers.

## Canonical entry points
- `app.py`: Primary FastAPI application server for local / OCI host deployment.
- `api/index.py`: Serverless entry point for Vercel deployment handler.
- `api/vercel_app.py`: Vercel-compatible FastAPI app implementation.
- `cloudflare-worker.js`: Cloudflare Edge Worker for traffic routing and automated failover.
- `auto_cleanup.py`: Storage maintenance script managed by `dada-cleanup.timer`.
- `validate_deployment.py`: Command-line system verification suite.
- `security.py`: Centralized security, IP geolocation, and file filter guard module.
- `chat_manager.py`: In-memory WebSocket manager and message queue for real-time chat.
- `dada.service`: Primary systemd service declaration.
- `Caddyfile`: Reverse proxy configuration for TLS termination and web server routing.
