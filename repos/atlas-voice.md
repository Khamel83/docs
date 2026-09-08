# LLM-OVERVIEW — atlas-voice
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
`atlas-voice` is a privacy-first AI voice pattern analyzer and system prompt generator. It extracts statistical and stylistic writing fingerprints (sentence length, vocabulary richness, function word frequencies, style markers, and sentence starters) from writing samples (email CSV exports, text/markdown files, and chat log JSON) and generates model-agnostic system prompts for AI assistants such as Claude, ChatGPT, Gemini, or local models.

The system enforces a privacy-by-design architecture: raw text ingested during import is processed strictly in temporary buffers (`data/imports/`) or in-memory, mapped into deterministic statistical models, persisted as anonymized voice patterns in a SQLite database (`data/atlas_voice.db`), and immediately deleted from disk. No original text or PII is ever retained in long-term storage.

The repository was rebranded from legacy voice matching projects (`c1bc0f9`). Historical multi-file implementations—including ARCHON integrations, strategic consulting frameworks, RAG systems, OOS workflows, room-two databases, airlock transfer logic, and nuclear safe room staging—have been retired into `_archive_old_project/`. The active production codebase is a clean 12-module Python package built under the ONE_SHOT framework, complemented by standalone LoRA fine-tuning scripts for local Llama 3.1 8B adaptation.

## Machine & Host Ownership
No multi-host information available from status probe or AGENTS.md; project contract `homelab.yaml` lists production host as unknown (`standby` state, managed_by unknown, unit `atlas-voice`).

## What is actually built

### 1. Core Python Package (`atlas_voice/`)
- **Data Models (`atlas_voice/models/`)**:
  - `pattern.py`: Defines `FunctionWord` (word string, percentage frequency), `StyleMarker` (category: `casual`, `formal`, `personal`, `technical`; phrase list), and `VoicePattern` dataclasses (`id`, `name`, `total_words`, `total_sentences`, `avg_sentence_length`, `vocab_richness`, `avg_word_length`, `function_words`, `style_markers`, `sentence_starters`, `source_description`, `created_at`). Provides `to_dict()` and `from_dict()` serialization.
  - `prompt.py`: Defines `VoicePrompt` dataclass (`id`, `pattern_id`, `name`, `context`, `prompt_text`, `created_at`) with dictionary conversion methods.
- **Service Layer (`atlas_voice/services/`)**:
  - `importer.py`: `ImportService` handles file and directory ingestion (`email` CSV via Pandas, plain `.txt`/`.md`, and JSON chat arrays). Cleans raw text into standard arrays and manages `data/imports/` temporary storage. Implements `clear_temp_files()` for privacy enforcement.
  - `analyzer.py`: `AnalyzerService` performs offline NLP pattern extraction using standard library regex tokenization and frequency analysis. Measures sentence counts, word lengths, Type-Token Ratio (vocabulary richness), top 20 function word frequencies (tracking usage of pronouns/articles), style marker classifications, and top 10 sentence starters. Returns a populated `VoicePattern`.
  - `generator.py`: `GeneratorService` maps a `VoicePattern` into a structured system prompt tailored to specific contexts (`professional`, `casual`, `creative`, `technical`). Synthesizes guidance sections covering writing style, language patterns, communication approach, context-specific directives, and verification checklists.
- **Storage Layer (`atlas_voice/storage/`)**:
  - `database.py`: `Database` class wrapping SQLite (`data/atlas_voice.db`). Schema contains `patterns` (`id`, `name`, `data` JSON blob, `created_at`) and `prompts` (`id`, `pattern_id` FK, `name`, `context`, `prompt_text`, `created_at`). Features context manager support, cascading deletion (`delete_pattern`), and pattern/prompt querying.
- **Web API Layer (`atlas_voice/api/`)**:
  - `app.py`: FastAPI application mounting REST routes and serving static web assets from `web/`. Endpoints include `/health`, `/api/import` (file upload + immediate pattern extraction + temp file purge), `/api/patterns` (list/get/delete), `/api/generate` (prompt generation), `/api/prompts` (list/get/download plain text), and `/api/privacy` (stored data inspection).
- **CLI Interface (`atlas_voice/cli/`)**:
  - `main.py`: Click CLI group (`atlas-voice`) exposing subcommands: `import` (`import_data`), `generate`, `list-patterns`, `show-pattern`, `list-prompts`, `export-prompt`, `privacy-check`, `clear-imports`, and `serve` (Uvicorn web server launcher).

### 2. Web Frontend (`web/`)
- `index.html`: Responsive single-page UI featuring tabbed navigation for file import, pattern management, prompt generation/viewing, and privacy auditing.
- `style.css`: Minimalist design layout and utility styling.
- `app.js`: Client-side logic for API communication, upload progress, pattern visualization, dynamic modal rendering, and clipboard actions.

### 3. Local LoRA Fine-Tuning Suite (Root Level)
- `prep_training_data.py`: Uses OpenRouter (`Gemini 2.5 Flash Lite`) to clean email CSV corpora (`clean_email_text`) and format instruction-output pairs into `training_data.jsonl`.
- `train_lora_local.py`: Leverages Unsloth (`FastLanguageModel`) for 4-bit QLoRA fine-tuning of Llama 3.1 8B on local GPU/Apple Silicon hardware, writing adapters to `omar-voice-lora`.
- `test_omar_model.py`: Loads the fine-tuned LoRA adapter with Unsloth 4-bit inference and evaluates prompt responses against chat templates.
- `start_lora_training.sh`: Shell wrapper validating dependencies, email data paths, and API keys before executing the dataset preparation, training, and testing pipeline.

### 4. Test Suite (`tests/`)
- `test_analyzer.py`: Unit test suite covering tokenizer accuracy, statistical calculations, and pattern extractions.
- `test_generator.py`: Tests prompt construction across all four context variants.
- `test_real_data_scale.py`: Research test script analyzing voice pattern convergence across time scales (1 week to 20 years, 18K+ emails, 8.6M+ words).

### 5. Retired Legacy Assets (`_archive_old_project/`)
- Archived code from the pre-rebuild era, including multi-stage pipeline stages (Room One, Airlock, Room Two), nuclear safe room file handlers, ARCHON framework bindings, OOS workflow integrators, and strategic consulting modules.

## Canonical entry points

- **CLI Application**: `atlas-voice` (installed via `pyproject.toml` entry point `atlas_voice.cli.main:cli`).
- **Web Application Server**: `atlas-voice serve [--host 127.0.0.1] [--port 8000]` (runs `uvicorn.run("atlas_voice.api.app:app")`).
- **FastAPI Instance**: `atlas_voice.api.app:app` (serves REST endpoints and web UI at `http://localhost:8000`).
- **Programmatic Python API**:
  - `from atlas_voice.services import ImportService, AnalyzerService, GeneratorService`
  - `from atlas_voice.storage import Database`
  - `from atlas_voice.models import VoicePattern, VoicePrompt`
- **LoRA Fine-Tuning Execution**:
  - `python3 prep_training_data.py --emails <path_to_csv> --output training_data.jsonl --num-samples 500`
  - `python3 train_lora_local.py --data training_data.jsonl --output omar-voice-lora`
  - `python3 test_omar_model.py --model omar-voice-lora`
  - `./start_lora_training.sh`
- **Test Runner**: `pytest` or `python3 -m pytest tests/`
