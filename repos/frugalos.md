# LLM-OVERVIEW — frugalos
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
Frugalos is a dual-component local-first LLM orchestration and personal AI assistant platform written in Python (>=3.11). It combines a CLI job execution engine (`frugalos`) for low-cost, schema-validated task execution with a FastAPI-based personal assistant service (`hermes`) providing local-first cost-aware routing, meta-learning, multi-modal capabilities (vision, audio), multi-tier caching, real-time streaming, and autonomous task execution.

## Machine & Host Ownership
- No multi-host infrastructure deployment is configured.
- `homelab.yaml` (schema `homelab.project/v1`) registers the project `frugalos` as a service owned by `homelab` in `development` lifecycle with `standby` monitoring state.
- Runtime configuration in `homelab.yaml` specifies production host as `unknown`, managed by `unknown`, and system unit as `frugalos`.

## What is actually built

### 1. Frugalos Job Engine (`frugalos/`)
- **CLI Interface (`frugalos/cli.py`)**: Typer and Rich-based CLI application exposing `frugal run`, `frugal receipts`, `frugal oracle`, and `frugal optimize-prompts`.
- **Job Runner (`frugalos/runner.py`)**: Executes single jobs configured by `--goal`, optional context (`--context`), JSON schema (`--schema`), and budget (`--budget-cents`).
  - **Local Execution ($T_1$)**: Invokes local Ollama models (`llama3.1:8b` for text, `qwen2.5-coder:7b` for code) defined in `frugalos/policy.yaml`.
  - **Consensus & Validation (`frugalos/validators/`)**: Performs $k$-sampling (`k_sample`), majority voting (`consensus.py`), and JSON schema validation (`schema.py`).
  - **Semantic Retry & Escalation**: Truncates context on validation failure and retries locally before checking `FRUGAL_ALLOW_REMOTE` and querying the oracle routing table (`frugalos/oracle/routing_table.json`) for remote $T_2$ escalation options.
- **Ledger & Cost Tracking (`frugalos/ledger.py`)**: SQLite storage engine at `out/receipts.sqlite`.
  - **Tables**: `receipts` (tracks timestamps, project, job IDs, latency, cost cents, tier, model path, template version, validation errors, and consensus votes), `prompt_templates` (versioning, metrics, A/B testing metadata), `prompt_examples` (few-shot example repository scored by quality and consensus).
- **Prompt Optimization (`frugalos/prompts/`)**: `template_manager.py`, `optimizer.py`, and `templates.py` handle template versioning, A/B testing, and dynamic few-shot example selection using local models.
- **Local Ollama Adapter (`frugalos/local/ollama_adapter.py`)**: HTTP adapter for sending prompts to Ollama (`http://localhost:11434`).

### 2. Hermes AI Assistant Platform (`hermes/`)
- **FastAPI Web Application & Launcher (`hermes/app.py`, `hermes/app_v2.py`, `run_hermes.py`)**: FastAPI application (v2.0.0) with GZip/CORS middleware, host/port CLI launcher (default `0.0.0.0:5000`), lifespan initialization for vision, audio, streaming, and multi-modal services, plus optional Redis L2 caching (`MultiTierCache`).
- **Local-First Cost Router (`hermes/routing/`)**:
  - Core engine: `LocalFirstRouter` (`router.py`), `LocalModelRunner` (`local_runner.py`), `CloudModelRunner` (`cloud_runner.py`), and `SessionManager` (`session.py`).
  - Routes prompts first to local Ollama models (`llama3.2:3b`, `llama3.1:8b`, `gemma3:latest`, `qwen2.5-coder:7b`, `deepseek-r1:8b`), calculates quality scores (target threshold 9.0/10), and prompts transparent cloud upgrade options (OpenRouter) when quality is insufficient.
  - Database (`hermes/routing/database.py` storing `hermes.db` or dedicated database): Maintains `sessions` and `tasks` tables tracking input/output tokens, actual vs predicted costs, model decisions, and latency.
  - REST API (`/api/v1/routing` via `api_routes.py`): Endpoints include `/process`, `/upgrade`, `/session/{session_id}`, `/session/{session_id}/end`, `/stats`, `/sessions/recent`, `/health`.
- **Unified Orchestrator & Job Queue (`hermes/orchestrator.py`, `hermes/job_queue.py`)**:
  - Singleton `HermesOrchestrator` managing background job worker threads (`JobQueue`), backend health monitoring (`health_monitor.py`), load balancing (`load_balancer.py`), failover management (`failover_manager.py`), and backend cost tracking (`cost_tracker.py`).
  - Database schema (`hermes/database.py`): Creates SQLite tables `jobs`, `job_events`, `conversations`, `conversation_messages`, `metalearning_patterns`, `metalearning_questions`, `metalearning_responses`, `backends`, `backend_configs`, `backend_performance`, `system_metrics`, `error_reports`, `notifications`, `monitoring_alerts`.
- **Meta-Learning & Autonomous Engine (`hermes/metalearning/`, `hermes/autonomous/`, `hermes/autonomous_dev/`)**:
  - Subsystems: `question_generator.py`, `pattern_engine.py`, `context_optimizer.py`, `adaptive_prioritizer.py`, `execution_strategy.py`, `metrics.py`.
  - Autonomous loops: `scheduler.py`, `suggestion_engine.py`, `context_automation.py`, `learning_optimizer.py`, and self-healing auto-optimizer components (`code_modifier.py`, `self_healing.py`, `auto_optimizer.py`).
- **Multi-Modal Services & Real-Time Streaming (`hermes/vision/`, `hermes/audio/`, `hermes/multimodal/`, `hermes/streaming/`)**:
  - `vision_service.py`: OCR and image understanding via OpenAI, Anthropic, and Google Vision APIs.
  - `audio_service.py`: Speech-to-text and text-to-speech via OpenAI, Azure Speech, AWS Polly, and Google Speech.
  - `multimodal_reasoner.py`: Combines visual, text, and audio context data.
  - `streaming_manager.py` & `streaming_orchestrator.py`: SSE/WebSocket streaming routes under `/api/v1/streaming`.
- **Security, Middleware & Personalization (`hermes/security/`, `hermes/middleware/`, `hermes/personalization/`)**:
  - Security: `auth_manager.py`, `auth_service.py`, `encryption.py`, `encryption_manager.py`, `compliance.py`, `threat_detector.py`.
  - Middleware: Rate limiting (`rate_limiter.py`), quota management (`quota_manager.py`), optimization middleware.
  - Personalization: `user_profiler.py`, `adaptive_conversations.py`, `contextual_awareness.py`, `personalized_generator.py`.
- **Subsystem API Routers (`hermes/routes/`)**: `model_routes.py`, `multimodal_routes.py`, `optimization_routes.py`, `personalization_routes.py`, `streaming_routes.py`.

### 3. Benchmarking, Testing & Database Files
- **Test Frameworks**:
  - `test_hermes_complete.py`: Master test suite executing 9 verification phases for Hermes v2.0.
  - `test_routing_mvp.py`, `test_debug.py`, `test_integration.py`: Validation scripts for local-first routing and integration.
- **Model Optimization & Characterization**:
  - `model_characterization_test.py`, `model_optimizer_suite.py`, `quick_cost_test.py`: Benchmark tools testing local Ollama models against OpenRouter cloud models.
  - Storage: `model_performance.db` and `cost_optimization_results.db`.

### 4. Dependencies & Packaging (`pyproject.toml`, `requirements.txt`)
- Build backend: `setuptools.build_meta`.
- Executable script mapping: `frugal = "frugalos.cli:app"`.
- Core dependencies: `typer`, `rich`, `requests`, `pydantic`, `jsonschema`, `pyyaml`, `sqlite-utils`, `Flask`, `Werkzeug`, `psutil`.

## Canonical entry points
- **Frugalos CLI Job Runner**:
  - Command: `frugal` (or `python3 -m frugalos.cli`)
  - Common invocations:
    - `frugal run --goal "Task goal description" [--project <project_name>] [--context <file_or_dir>] [--schema <schema.json>] [--budget-cents <cents>]`
    - `frugal receipts [--project <project_name>]`
    - `frugal oracle [--show] [--free]`
    - `frugal optimize-prompts`
- **Hermes FastAPI Application Launcher**:
  - Script launcher: `python3 run_hermes.py [--host 0.0.0.0] [--port 5000] [--debug] [--init-db]`
  - Direct ASGI server: `uvicorn hermes.app:app --host 0.0.0.0 --port 5000` (or `hermes.app_v2:app`)
- **Local-First Router API**:
  - Python interface: `from hermes.routing.router import LocalFirstRouter`
  - REST endpoints: `POST /api/v1/routing/process`, `POST /api/v1/routing/upgrade`, `GET /api/v1/routing/stats`
- **Unified Orchestrator**:
  - Python interface: `from hermes.orchestrator import get_orchestrator`
- **Testing & Model Optimization Suites**:
  - Master test suite: `python3 test_hermes_complete.py`
  - Routing test: `python3 test_routing_mvp.py`
  - Model optimizer benchmarks: `python3 model_optimizer_suite.py`
