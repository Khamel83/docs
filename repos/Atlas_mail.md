# LLM-OVERVIEW — Atlas_mail
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
Atlas_mail is a specialized email and message data ingestion, pre-processing, vectorization, and AI style-cloning platform. It ingests multi-channel personal and work communications—including Slack thread exports, macOS iMessage databases, Google Takeout/Outlook email archives, and media attachments—normalizing them into a structured database schema (`schema.sql`) and a vectorized dataset.

The core functional subsystem is the **Omarizer** style-cloning engine stored under `1shot/style_cloning/`. Following empirical evaluation where fine-tuned LoRA models produced marginal gains and failed promotion gates, fine-tuning was retired in favor of a prompt-based architecture. The repository houses the raw corpus, cleaned text pairs, style profile codecs, vector indices, RAG retrieval tools, FastAPI microservices, evaluation frameworks, and portable system prompt definitions (`OMARIZER_PROMPT.md` and Codex skills).

## Machine & Host Ownership
- **Primary Host**: `macmini` (macOS host environment).
- **Execution & Storage Context**:
  - Active runtime scripts (`run_on_macmini.sh`), launchd daemons (`com.khamel83.emailsync.plist`), and macOS iMessage importers rely on local macOS paths and system APIs.
  - External SSD paths on `macmini` (untracked in git):
    - `/Volumes/2TB_SSD/dev/style_cloning/` — Training run outputs, adapters, and pipeline configs.
    - `/Volumes/2TB_SSD/dev/omar_vector_store/` — ChromaDB vector database (~110K vectors).
    - `/Volumes/2TB_SSD/dev/mlx_training_env/` — Python environment with MLX/`mlx_lm` dependencies.
- **Status Probe**: Disabled (`JANITOR_RUN_STATUS_PROBE=1` required to execute `scripts/status.py`). No additional multi-host topology is defined in context.

## What is actually built

### 1. Style Cloning & Corpus Subsystem (`1shot/` & `1shot/style_cloning/`)
- **Data Archive (`1shot/style_cloning/`)**: Single source of truth for style cloning data and prompts.
  - `raw/`: Untracked/irreplaceable Slack logs, personal writing, and message archives.
  - `processed/`: Cleaned training pairs and dataset splits (62K Slack pairs, 3.8K email thread pairs, 17K iMessage pairs).
  - `prompts/`: `OMARIZER_PROMPT.md` (standalone, portable system prompt for any LLM), `SKILL.md` (Codex skill spec in `1shot/omarizer_skill/`), and reference style codec profiles.
- **Pipeline & Dataset Generators (`1shot/`)**:
  - `build_vector_store.py`: Ingests corpus into a ChromaDB vector store (110K vectors).
  - `build_slack_threads.py`: Extracts and pairs conversational threads from Slack exports (62K pairs).
  - `build_email_threads.py`: Reconstructs email threads into turn pairs (3.8K pairs).
  - `build_imessage_dataset.py`: Formats raw iMessage sqlite logs into training pairs (17K pairs).
  - `validate_training_data.py`: Validates schemas, merges multi-channel pairs, and deduplicates.
  - `build_omarizer_profile.py`: Analyzes corpus to generate style codec profiles.
  - `rag_retriever.py`: RAG search interface querying ChromaDB.
- **API Microservices (`1shot/`)**:
  - `omarizer_api.py`: FastAPI server delivering Omarizer style conversions on port 8100.
  - `adapter_api.py`: FastAPI server providing model adapter interfaces on port 8101.
- **Evaluation & Benchmarking (`1shot/`)**:
  - `build_eval_set.py`: Constructs evaluation splits, automated style scoring, and blind A/B test tools.
  - `evaluate_checkpoints.py`: Measures perplexity (PPL) and style match metrics across model checkpoints.
- **Retired Fine-Tuned Subsystem**:
  - LoRA fine-tuning (MLX/Qwen3 pipeline) was evaluated but retired after failing promotion gates. Training scripts remain for historical reference; active production relies strictly on prompt-based style cloning.

### 2. Communications Ingestion & Preprocessing Subsystem
- **Relational Schema**:
  - `schema.sql`: SQL database definition for storing imported emails, chat messages, contact profiles, and attachment metadata.
- **Email Ingestion**:
  - `email_importer.py` & `configure_addresses.py`: Ingests Outlook exports and Google Takeout archives into SQLite.
  - `DATA_INGESTION_GUIDE.md` & `GOOGLE_TAKEOUT_OUTLOOK_GUIDE.md`: Operational guides for email ingestion.
- **iMessage Extraction**:
  - `import_imessages.py`, `import_all_messages.sh`, `complete_messages_import.sh`: Connects to macOS `chat.db`, extracts chat threads, maps attachments, and imports into SQLite.
- **Slack Parsing**:
  - `extract_slack_messages.py`: Parses Slack JSON export files, preserving channel structure and thread relationships.
- **Unified Preprocessing**:
  - `extract_all_value.py` & `comprehensive_preprocessor.py`: Sweeps all raw channels (Slack, iMessage, Email), executes entity extraction, normalizes text, and produces dataset pairs.
  - `preprocessing_plan.md`: Processing architecture and cleanup specification.
- **Attachment Management**:
  - `mail_inventory.py`: Catalogs email data balances and message statistics.
  - `organize_attachments.py` & `copy_all_attachments.sh`: Extracts, renames, and structures message binary attachments into organized directories.

### 3. Service Daemons & Homelab Integration
- `sync_service.sh` & `com.khamel83.emailsync.plist`: Shell sync routine and macOS `launchd` service daemon for ongoing background mail synchronization on `macmini`.
- `homelab.yaml`: Project contract and infrastructure metadata for Homelab environment management.
- Execution Utilities: `run_on_macmini.sh` (host command launcher), `fix_and_complete.sh`, and `install.sh` (setup/maintenance scripts).

## Canonical entry points
- **Style Cloning (Prompt & Services)**:
  - `1shot/style_cloning/prompts/OMARIZER_PROMPT.md`: Standalone system prompt for Omarizer style transformation across standard LLMs.
  - `1shot/omarizer_skill/`: Codex skill implementation referencing prompt definitions.
  - `1shot/omarizer_api.py`: FastAPI server entry point (Port 8100).
  - `1shot/adapter_api.py`: FastAPI server entry point (Port 8101).
- **Corpus & Data Generation**:
  - `1shot/build_vector_store.py`: ChromaDB embedding generation tool.
  - `1shot/build_slack_threads.py`, `1shot/build_email_threads.py`, `1shot/build_imessage_dataset.py`: Extractor scripts for Slack, email, and iMessage data.
  - `1shot/validate_training_data.py`: Dataset consolidation and deduplication runner.
  - `1shot/build_omarizer_profile.py`: Style codec extraction entry point.
  - `1shot/build_eval_set.py` & `1shot/evaluate_checkpoints.py`: Benchmarking and checkpoint evaluation entry points.
- **Ingestion & Sync Workflows**:
  - `email_importer.py`: Command-line tool for importing email archives.
  - `import_imessages.py` / `complete_messages_import.sh`: iMessage database extraction entry point.
  - `extract_slack_messages.py`: Slack archive extraction entry point.
  - `comprehensive_preprocessor.py` / `extract_all_value.py`: Integrated multi-channel pre-processing and data extraction entry point.
  - `sync_service.sh` / `com.khamel83.emailsync.plist`: Background mail sync service daemon.
  - `run_on_macmini.sh`: Shell wrapper for executing operations on host `macmini`.
