# LLM-OVERVIEW — boys
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
A private, single-tap voice message exchange application built for two children (Ramy and Haroun) living in different cities. The application features a two-panel web interface with zero authentication or account requirements, permitting immediate audio recording and cross-feed message delivery. Audio is transcribed asynchronously on local CPU using `faster-whisper`, and notifications with transcribed text are dispatched to a configured Telegram chat.

## Machine & Host Ownership
No multi-host information is available; configuration targets a single host deployment environment (service directory `/home/ubuntu/github/boys`, run by system user `ubuntu`).

## What is actually built
- **Backend API Service (`app.py`)**:
  - FastAPI application running on Uvicorn (port 8003).
  - Enforces reciprocal sender-recipient mapping (`SENDERS = {"ramy": "haroun", "haroun": "ramy"}`).
  - Endpoints:
    - `GET /`: Renders `templates/index.html` via Jinja2 templates.
    - `GET /health`: Health probe returning `{"status": "ok"}`.
    - `POST /record/{sender}`: Receives audio upload via multipart form data (`UploadFile`), stores the file in `UPLOAD_DIR` (`/home/ubuntu/uploads/boys`) named `{uuid}.{ext}`, inserts a message record into SQLite, and triggers background transcription/notification.
    - `GET /messages/{recipient}`: Queries database for the latest 50 messages sent to the specified recipient.
    - `GET /audio/{filename}`: Serves recorded audio files from `UPLOAD_DIR` with dynamic MIME type resolution (`audio/mp4`, `audio/ogg`, or default `audio/webm`).

- **Database Layer (`messages.db`)**:
  - SQLite database initialized automatically via `init_db()`.
  - Table `messages`:
    - `id TEXT PRIMARY KEY`: UUID v4 identifier.
    - `sender TEXT NOT NULL`: Message author (`ramy` or `haroun`).
    - `recipient TEXT NOT NULL`: Target recipient (`haroun` or `ramy`).
    - `filename TEXT NOT NULL`: Stored audio filename (e.g. `{msg_id}.webm` or `{msg_id}.mp4`).
    - `timestamp TEXT NOT NULL`: ISO 8601 UTC timestamp.
    - `notified INTEGER DEFAULT 0`: Telegram notification marker.

- **AI Transcription & Telegram Notifications**:
  - `faster-whisper`: Lazy-loads `WhisperModel("small", device="cpu", compute_type="int8")`.
  - Runs transcription asynchronously in a thread executor (`transcribe_async`).
  - `send_telegram`: Dispatches HTML-formatted message notifications containing sender, recipient, timestamp, and transcript to Telegram via `aiohttp` (`https://api.telegram.org/bot{TELEGRAM_BOT_TOKEN}/sendMessage`).

- **Frontend Client (`templates/index.html`)**:
  - Vanilla HTML5/CSS3/JavaScript SPA (~490 lines) using Google Font `Nunito`.
  - Dual-panel layout (Ramy in orange, Haroun in blue) with a 6-theme picker (Safari, Space, Dinosaurs, Ocean, Pirates, Ninjago).
  - Browser MediaRecorder integration with MIME type detection supporting iOS Safari (`audio/mp4`) and Chrome/Android (`audio/webm`).
  - Auto-polls recipient feeds every 10 seconds.

- **Infrastructure & Maintenance (`boys.service`, `backup.sh`, `homelab.yaml`)**:
  - `boys.service`: Systemd service unit executing `uvicorn app:app --host 0.0.0.0 --port 8003 --workers 1` under environment file `/home/ubuntu/github/boys/.env`.
  - `backup.sh`: Daily backup script archiving SQLite database and uploads into `/home/ubuntu/backups/boys/boys-backup-YYYY-MM-DD.tar.gz` with 30-day retention (`-mtime +30`).
  - `homelab.yaml`: Project metadata contract (`schema: homelab.project/v1`, project `boys`).

## Canonical entry points
- `app.py`: Backend entry point containing database initialization, route handlers, transcription worker, and Telegram sender.
- `templates/index.html`: Primary user interface entry point rendered at path `/`.
- `boys.service`: Systemd unit file for process daemonization on port 8003.
- `backup.sh`: Operational script for database and upload backups.
- `homelab.yaml`: Homelab project definition contract.
