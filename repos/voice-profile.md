# LLM-OVERVIEW — voice-profile
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
Prompt engineering framework that constructs personal "voice profiles" from authentic spoken transcripts (`speech.md`) and email archives to style AI model outputs. Uses statistical corpus analysis and NLP pattern extraction rather than model fine-tuning to generate system prompts reflecting an individual's vocabulary, sentence structure, emotional tone, and technical communication style. Project lifecycle is `development` in monitoring status `standby`.

## Machine & Host Ownership
Per `homelab.yaml`, the target production host and managing infrastructure are classified as `unknown` with systemd unit `voice-profile`. No multi-host deployment configuration or remote server specs exist in the repository.

## What is actually built
- **Data Ingestion Pipeline (`src/data_processor.py`, `src/email_processor.py`)**:
  - `data_processor.py`: Parses transcript corpora into `SpeechEntry` dataclasses (tracking entry ID, float timestamps, content, word count, context type, technical content flags, and sentiment scores). Aggregates metrics into `VoiceData` containers.
  - `email_processor.py`: `EmailProcessor` streams CSV email exports with unconstrained field limits (`sys.maxsize`). Converts records into `EmailEntry` objects capturing subject, content, human authorship boolean (`is_human`), recipient count, technical indicators, and sentiment scores.
- **NLP & Corpus Analysis Engines (`src/voice_analyzer.py`, `src/comprehensive_analyzer.py`)**:
  - `voice_analyzer.py`: Utilizes `nltk` (tokenizers, stopwords) and `textstat` readability metrics to extract `VoicePatterns` covering vocabulary frequency distributions, sentence length/complexity stats, and context-specific phrasing.
  - `comprehensive_analyzer.py`: `ComprehensiveAnalyzer` unifies speech and email corpora from SQLite (`~/.voice_profile/voice_data.db`) into `ComprehensiveVoiceProfile` structures mapping vocabulary, sentence metrics, emotional tone, and technical indicators.
- **Out-of-System (OOS) Integration (`src/oos_integration.py`)**:
  - `OOSVoiceIntegration`: Bridges local voice data (`~/.voice_profile/voice_data.db`) with external OOS system state (`~/.oos/oos.db`), generating `OOSVoiceEnhancement` objects to prepend voice traits to standard system requests.
- **Voice Similarity Evaluation (`src/voice_tester.py`)**:
  - `VoiceTester`: Benchmarking suite comparing generated AI text against expected profile characteristics via string matching (`difflib`) and criteria scoring. Persists `TestResult` metrics against defined `TestCase` instances.
- **CLI Utilities (`bin/`)**:
  - `bin/voice-init`: Environment and database setup script.
  - `bin/voice-import`: Ingests speech transcripts (`--speech`) or email dumps (`--emails`).
  - `bin/voice-generate`: Generates reusable profile prompts (`--name`).
  - `bin/voice-ask`: Formats queries using a target voice profile (`--voice`).
- **Dependencies (`requirements.txt`)**: `pandas`, `click`, `rich`, `nltk`, `textstat`.

## Canonical entry points
- `./bin/voice-init`: Initializes local storage and environment.
- `./bin/voice-import`: Data import entry point for speech and email datasets.
- `./bin/voice-generate`: Profile compilation and prompt generation CLI.
- `./bin/voice-ask`: Prompt formatting wrapper for AI model interactions.
- `src/comprehensive_analyzer.py`: CLI/module for unified multi-source corpus analysis.
- `src/oos_integration.py`: Adapter layer interfacing with `~/.oos/oos.db`.
- `src/voice_tester.py`: Evaluation runner for testing AI output against voice profiles.
- `homelab.yaml`: Homelab project contract (`homelab.project/v1`).
