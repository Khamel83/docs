# LLM-OVERVIEW — penny
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is

Penny is a local-first voice capture, transcription, and routing middleware system. It ingests voice memos from Apple Watch and iCloud Voice Memos (`CloudRecordings.db`), disk backlogs, and authorized HTTP webhook payloads, preserves them in local content-addressed storage, transcribes audio using a pinned offline MLX Whisper model (`whisper-large-v3-turbo`), evaluates transcript quality with deterministic gates, and durably dispatches results to downstream consumers: receipt-backed local Apple Notes and Reminders, Slack (Block Kit v2 and plain-text continuations), Maya v2 intake (durable outbox with UTC ISO-8601 payloads and drop receipts), and Hermes webhooks. It also manages immutable audio object archiving with iCloud mirror sync, versioned local-first database/audio backups with scratch verification, and metadata-only read-only readiness probes via Doctor.

## Machine & Host Ownership

No multi-host information is available in the live status probe output or `AGENTS.md`.

## What is actually built

### 1. Canonical State & Schema (`transcript_log.py`, `config.py`)
`transcript_log.py` owns the additive SQLite database schema persisted at `~/.penny/transcripts.db`. SQLite handles concurrent access via timeout locking and explicit transaction boundaries.

* **`transcripts`**: Core canonical ledger containing `id`, `filename`, `file_hash`, `transcript`, `created_at`, `routed`, `routing_result`, `quality_status` (`passed`, `needs_review`), `quality_detail`, `quality_score`, `transcribe_duration_seconds`, `audio_duration_seconds`, `transcription_attempt_count`, `source` (`voice_memos`, `webhook`), `source_root`, `stage_status`, `archived_at`, `archive_status`, `archive_path`, `mirror_path`, `mirror_status`, `maya_delivery_status`, `maya_delivery_attempt_count`, `maya_delivery_receipt`, `maya_delivery_error_code`, `maya_delivery_next_attempt_at`, `maya_delivery_claim_token`, `maya_delivery_claim_owner`, `maya_delivery_claim_expires_at`.
* **`voice_memos`**: Recording discovery ledger tracking `Z_PK` (`recording_pk`), `audio_path`, `discovered_at`, `duration_seconds`, `file_seen_at`, `recording_timestamp`, `status` (`waiting_for_file`, `seen`, `transcribed`, `routed`, `retryable`, `failed_terminal`), `linked_transcript_id`, `retry_count`, `next_retry_at`, `last_error_code`.
* **`watermarks`**: Source discovery cursor tracking `source` and `last_discovered_id`. Advances the `voice_memos` cursor only after a durable `voice_memo_ingest` transaction upsert.
* **`slack_deliveries`**: Outbox for Slack message deliveries tracking `id`, `transcript_id`, `delivery_plan` (`block_kit_v2`, `legacy_top_level_v1`), `status` (`pending`, `in_flight`, `sent`, `failed`, `reconciliation_required`), `claim_token`, `claim_owner`, `claim_expires_at`, `attempt_count`, `next_attempt_at`, `last_error_code`, `slack_ts`, `slack_channel_id`.
* **`quality_failure_deliveries`**: Outbox tracking body-free quality failure metadata projections dispatched to the Maya ledger channel.
* **`apple_effects`**: Receipt-backed local effect ledger tracking `effect_key` (SHA-256 of `transcript_id`, `effect_type`, `normalized_target_name`, `payload_hash`), `transcript_id`, `effect_type` (`note`, `reminder`), `target_name`, `normalized_payload_hash`, `state` (`reserved`, `in_flight`, `uncertain`, `succeeded`, `failed`, `quarantined`), `provider_id`, `actual_target`, `attempt_count`, `claim_token`, `claim_owner`, `claim_expires_at`, `last_error_code`.
* **`archive_deliveries`**: Outbox tracking content-addressed immutable audio packaging and iCloud mirror publication.
* **Transcription Identity (`config.py`)**: Durable identity pinned to repository `mlx-community/whisper-large-v3-turbo` at revision `a4aaeec0636e6fef84abdcbe3544cb2bf7e9f6fb` stored locally at `~/.penny/models/whisper-large-v3-turbo/a4aaeec0636e6fef84abdcbe3544cb2bf7e9f6fb`. Requires local receipt verification and `HF_HUB_OFFLINE=1`.

### 2. Ingestion & Daemon Orchestration (`watcher.py`, `core.py`, `ingress_auth.py`, `webhook/server.py`)
* **Discovery & Polling Daemon (`watcher.py`)**: Runs main loop scanning `CloudRecordings.db` for new `Z_PK` rows exceeding the durable watermark, reading `~/.penny/last_pk.txt` compatibility mirror, and scanning disk backlog in `~/Library/Application Support/com.apple.voicememos/Recordings` (files created < 24h ago, size < 50MB).
* **Daemon Readiness & Recycling**: Verifies process health for `VoiceMemos` and `com.apple.VoiceMemos.SpotlightExtension`. If `VoiceMemos` fails AppleScript responsiveness probes for 3 consecutive checks, recycles the process via AppleScript.
* **Ingest Passes (`_process_ingest_pass`)**: Coordinates sequential batch pipeline: (1) DB recording discovery & metadata upsert, (2) retry recordings waiting for files, (3) disk backlog scan & processing, (4) unlinked source retries, (5) pending route retries, (6) Slack outbox delivery, (7) Maya outbox delivery, (8) Archive outbox processing, and (9) Archive backfill and published mirror verification.
* **Webhook Server (`webhook/server.py`, `ingress_auth.py`)**: Flask web server on port 8080. `/health` provides liveness, `/ready` invokes Doctor readiness checks, and `/ingest` accepts audio files or text note payloads. Authenticated using `authorize_bearer` comparing `PENNY_WEBHOOK_SECRET` in constant time; enforces a 64KB (`MAX_INGEST_TEXT_BYTES`) limit on text ingestion.

### 3. Transcription & Quality Control (`transcript_quality.py`, `classifier.py`)
* **MLX Whisper Execution**: Transcribes audio using `mlx-whisper` using pinned local weights verified against `.penny-committed` marker and SHA-256 manifests (`model.safetensors`, `config.json`, `tokenizer.json`).
* **Quality Evaluation (`transcribe_with_quality`)**: Runs up to two transcription attempts (primary vs fallback options). `evaluate_transcript` strips control tokens (`<|...|>`), checks for token repetition (max 3 consecutive, excluding natural spoken emphasis `"no"`), and checks for low-diversity suffix loops (window of 20 tokens, max 2 unique tokens). Transcripts failing quality gates receive `quality_status = "needs_review"` and trigger body-free quality failure outbox projections.
* **Classifier (`classifier.py`)**: Classifies transcripts into categories (`groceries`, `errands`, `home`, `health`, `work`, `kids`, `inbox`) or content types (`action_items`, `long_note`, `unclear`). Direct OpenRouter classification is legacy/transitional fallback.

### 4. Durable Routing & Outbox Pipelines
* **Apple Notes & Reminders (`apple_effects.py`, `reminders.py`)**: Receipt-backed local effects. Claims rows with 120s CAS lease (`APPLE_EFFECT_LEASE_SECONDS`), max 5 attempts with backoff (0, 1, 5, 30, 300s). `reminders.py` executes `osascript` to create Notes/Reminders embedded with marker strings (`penny-effect:<hash>`), performing read-after-write verification to return a `ProviderReceipt`.
* **Slack Delivery (`slack_delivery.py`)**: Outbox processor formats transcripts into Block Kit v2 blocks or sectioned plain-text continuations. Claims rows with 30s lease (`SLACK_CLAIM_LEASE_SECONDS`), generating deterministic `client_msg_id` via UUIDv5. Also processes quality failure projections (`process_pending_quality_failure_deliveries`) to notify the Maya ledger.
* **Maya v2 Intake (`maya_delivery.py`)**: Builds UTC ISO-8601 envelopes (`build_maya_v2_envelope`). Claims pending rows with 120s lease (`MAYA_CLAIM_LEASE_SECONDS`), max 20 attempts, max 7-day age window. Posts to Maya transcript URL with Bearer token authentication, requiring an exact v2 `drop_id` receipt. Employs exponential backoff (30s to 1800s) and terminalizes capped/stale rows to dead-letter state.
* **Hermes Webhook (`core.py`)**: Best-effort notification delivery via `PENNY_HERMES_WEBHOOK_SECRET` when action items are extracted.

### 5. Archiving & Local-First Backups (`archive.py`, `backup.py`)
* **Immutable Archive (`archive.py`)**: Stages audio content-addressed (`.m4a`/`.wav` named by SHA-256 hash) in private `0o700` local store (`~/.penny/archive/objects`). Generates Markdown transcript file and publishes atomically to iCloud mirror (`publish_archive`), writing JSON manifest last. `validate_archive` and `validate_local_mirror_receipt` enforce strict hash matching across the audio, markdown, and manifest trio.
* **Local-First Backups (`backup.py`)**: Creates versioned backup sets (`create_backup_set`) identified by timestamp (`YYYYMMDDTHHMMSSZ`). Snapshots SQLite database using online backup API with integrity checks and hard-links/copies immutable audio objects.
* **Scratch Verification (`verify_backup_set`)**: Restores and validates backup sets inside an isolated private scratch directory (`_safe_scratch`), checking file sizes, SHA-256 hashes, catalog specs, and database structural integrity via `PRAGMA integrity_check`. Performs zero live database writes during verification. Enforces a 90-day retention plan (`plan_retention`). Rejects cloud-synced paths (`Mobile Documents`, `CloudDocs`) and symlinks for backup roots to prevent sync corruption.

### 6. Readiness Probes & Operations (`doctor.py`, `scripts/`)
* **Doctor (`doctor.py`, `scripts/penny_doctor.py`)**: Read-only metadata-only audit tool. Returns exit codes `0` (Ready), `1` (Degraded), or `2` (Unready). Probes 10 components: `sqlite`, `voice_memos`, `archive`, `transcription`, `apple_effects`, `maya`, `slack`, `services`, `backup`, and `ingress`. Zero-leak guarantee: never reports audio/transcript content, raw file paths, URLs, secrets, TCC databases, process IDs, or raw error strings; redacts exceptions to standard class names and details to boolean/integer counters. Tracks launchd service startup revision via `_STARTUP_SOURCE_REVISION`.
* **Management Scripts (`scripts/`)**:
  * `backup_penny.py`: Production script to trigger backup set creation.
  * `verify_penny_backup.py`: CLI wrapper for scratch backup verification.
  * `pin_whisper_model.py`: Validates model weights and writes `.penny-committed`.
  * `export_transcripts.py`: Generates readable JSON/Markdown exports (readable aids only).
  * `re_evaluate_quality_review.py`: Re-evaluates quality review rows against active policy.
  * `replay_maya_delivery.py`: Manually reopens failed/dead-lettered Maya outbox rows.
  * `trust_check.py`: Audits local dependencies and environment validity.

## Canonical entry points

* **`watcher.py` (`main()`)**: Main background daemon polling `CloudRecordings.db`, managing disk backlogs, executing MLX Whisper transcription, and driving outbox delivery passes for Slack, Maya, and Archive.
* **`webhook/server.py` (`app.run()`)**: HTTP service entry point serving `/ingest`, `/health`, and `/ready`. Launched via launchd plist `com.penny.webhook.plist`.
* **`doctor.py` / `scripts/penny_doctor.py` (`run_doctor()`)**: CLI and module entry point for read-only readiness probes.
* **`scripts/backup_penny.py`**: Production entry point for creating versioned immutable local backup sets.
* **`scripts/verify_penny_backup.py`**: Entry point for scratch-isolated verification of existing backup sets.
* **`scripts/pin_whisper_model.py`**: CLI utility for verifying local model files and pinning `.penny-committed`.
* **`tasks_poller.py` (`main()`)**: Background daemon polling Google Tasks API to ingest items into Apple Reminders/Notes.
* **`scripts/replay_maya_delivery.py`**: Operator CLI to manually reset and replay failed or dead-lettered Maya outbox rows.
* **`scripts/re_evaluate_quality_review.py`**: Operator CLI to re-evaluate retained transcript rows flagged for quality review.
* **Launchd Plist Templates (`launchd/`)**:
  * `com.penny.watcher.plist.template`: Service template for `watcher.py`.
  * `com.penny.webhook.plist.template`: Service template for `webhook/server.py`.
  * `com.penny.tasks.plist.template`: Service template for `tasks_poller.py`.
  * `com.penny.export.plist.template`: Scheduled job template for transcript exports.
