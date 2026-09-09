# LLM-OVERVIEW — oneshot
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
`oneshot` is a multi-model orchestration control plane and task delegation harness designed for Claude Code. It allows Claude to operate as a high-level planner and reviewer while routing bounded execution, coding, research, writing, and maintenance tasks across a matrix of external workers (Codex, Gemini CLI, GLM Claude / ZAI, Manus, OpenCode Go, OpenRouter free pool). The architecture enforces strict separation of planning from execution, manages task lifecycles within isolated Git worktrees, captures structured execution traces, runs background intelligence via zero-cost Janitor jobs, handles search via the Argus homelab broker, and manages cross-session repo memory and decision provenance.

## Machine & Host Ownership
Based on configuration (`homelab.yaml`, `config/workers.yaml`, `config/search.yaml`), `oneshot_cli/doctor_cmd.py`, and `AGENTS.md`:
- **Local Host (`mba` / `localhost`)**: Primary execution node running Claude Code planner, local CLI commands (`bin/oneshot`), local worker harnesses (`claw_code`, `glm_claude` via ZAI GLM-5-turbo, `ocg_minimax` / `ocg_api` via OpenCode Go, `manus` direct API, `free` via OpenRouter free pool), session event recording, and Git worktree creation (`../oneshot-worktrees/`). `homelab.yaml` defines production runtime on `mba` as a pull-on-demand service.
- **Remote SSH Hosts**:
  - `oci` (`oci-ts`): SSH-accessible remote host configured for `claude_code` planner execution.
  - `macmini` (`macmini-ts`): SSH-accessible remote worker host executing `opencode` or `gemini_cli`.
  - `homelab` (`homelab-ts`): SSH-accessible remote worker host executing `opencode`.
- **Homelab Search Service**: `http://100.112.130.100:8270` — Argus search broker host accessible over Tailscale.
- Status probe: Disabled (`JANITOR_RUN_STATUS_PROBE=1` required to run `scripts/status.py`).

## What is actually built

### Core Router (`core/router/`)
- `task_schema.py`: Defines task dataclasses and enumeration schemas:
  - `TaskClass`: Enum covering `plan`, `research`, `search_sweep`, `implement_small`, `implement_medium`, `test_write`, `review_diff`, `adversarial_review`, `doc_draft`, `summarize_findings`, `janitor_summarize`, `janitor_extract`.
  - `TaskCategory`: `coding`, `research`, `writing`, `review`, `general`.
  - `TaskSize` (`small`, `medium`, `large`), `RiskLevel` (`low`, `medium`, `high`).
- `plan_schema.py`: Structured plan schema for machine-readable blueprints: `Plan`, `PlanStep` (`StepAction`: `explore`, `add_tests`, `implement`, `refactor`, `verify`, `document`, `configure`), and `VerifyStep` (`VerifyType`: `test`, `lint`, `typecheck`, `acceptance`, `diff_review`).
- `lane_policy.py`: Core routing engine loading `config/lanes.yaml`. Evaluates task class and category to determine lane assignment, worker pools, preferred worker ordering, reviewer harness, fallback lane for escalation, and search backend.
- `model_registry.py`: Model capability registry loading `config/models.yaml`. Queries model attributes (`can_plan`, `can_review`, lane mapping).
- `resolve.py`: Router CLI resolver emitting JSON directives (`python3 -m core.router.resolve --class <class> [--category <cat>]`).

### System Configurations (`config/`)
- `lanes.yaml`: Defines 5 operational lanes:
  - `premium`: Planner `claude_code`, reviewer `claude_code`, worker pool `[claude_code, codex]`, max parallel 2.
  - `balanced`: Worker pool `[codex, gemini_cli]`, fallback `premium`, max parallel 3.
  - `cheap`: Worker pool `[codex, manus, gemini_cli, glm_claude]`, fallback `balanced`, max parallel 3.
  - `research`: Worker pool `[manus, gemini_cli, codex]`, search backend `argus`, fallback `balanced`.
  - `janitor`: Exclusively `[free]` (OpenRouter), no reviewer (`null`), fallback `null`, max parallel 1, retry limit 3.
- `workers.yaml`: Host, transport (`local`, `ssh`), harness (`claude_code`, `opencode`, `claw_code`, `direct_api`, `openrouter_api`), provider, and expiration configs for workers (`local`, `oci`, `macmini`, `homelab`, `claw`, `glm_claude`, `ocg_minimax`, `ocg_api`, `free`, `manus`).
- `models.yaml`: Capabilities and lane mappings per model.
- `providers.yaml`: API provider endpoint definitions.
- `search.yaml`: Argus client config (`base_url: http://100.112.130.100:8270`, `api_key_env: ARGUS_API_KEY`). Search modes (`discovery`, `grounding`, `recovery`, `research`) and mode aliases (`cheap`, `precision`).

### Dispatch Runner (`core/dispatch/`)
- `run.py`: Multi-worker parallel dispatch runner using Python `ProcessPoolExecutor`. Builds prompts, executes worker CLI/API commands, handles secrets redaction, and outputs execution manifests into `1shot/dispatch/{id}.md` or task directories.
- Trace System: Generates structured trace bundles in `eval/traces/{date}/{task_class}-{HHMMSS}-{worker}/`:
  - `trace.json`: Full structured execution trace (primary machine artifact).
  - `prompt.md`: Exact rendered prompt delivered to worker.
  - `output.raw`: Raw output capture from worker.
  - `manifest.md`: Human-readable summary derived from `trace.json`.
- `direct_api.py`: Direct OpenAI/OpenCode Go API caller (`call(base_url, model, task_file)`) for headless tasks without shell access. Automatically redacts API keys matching pattern `sk-*` / `key-*`.

### Background Intelligence Janitor (`core/janitor/`)
- `recorder.py`: `SessionRecorder` appends event stream (`user_request`, `action_taken`, `file_read`, `file_written`, `decision`, `blocker`, `discovery`, `commit`, `session_start`, `session_end`, `summary`, `error`) to `.janitor/events.jsonl` and builds searchable SQLite index `.janitor/intelligence.db`.
- `jobs.py`:
  - Pure-compute jobs: `detect_project_type`, `detect_test_gaps`, `scan_code_smells`, `detect_config_drift`, `build_dependency_map`, `detect_document_staleness`, `detect_orphan_documents`, `detect_size_outliers`, `detect_cross_references`, `detect_missing_frontmatter`.
  - LLM background jobs (OpenRouter free pool): `summarize_session`, `mine_patterns`, `generate_onboarding`, `enrich_commits`, `evaluate_task_sufficiency`, `generate_pending_tasks`, `review_pending_tasks`.
  - Session lifecycle: `run_session_start()` (injects compute context) and `run_session_end()` (triggers asynchronous LLM summary jobs).
- `worker.py`: OpenRouter free tier client (`call_free`, `extract_structured`) with multi-tier fallback model lists (`MODEL_TIERS`), rate-limiting (1000 daily, 20/min), and usage tracking in `.janitor/usage.jsonl`.
- `inbox.py` & `digest.py`: Message queue management and project intelligence digest generation.
- `hooks/`: Lifecycle bash hooks (`context.sh`, `pre-compact.sh`, `record.sh`, `session-end.sh`) integrating Janitor into Claude Code events.

### Search Integration (`core/search/`)
- `argus_client.py`: HTTP API client for homelab Argus search broker (`http://100.112.130.100:8270`). Resolves API keys via config, `ARGUS_API_KEY` env, or `secrets get ARGUS_API_KEY`.
- Functions: `search()`, `health()`, `recover_article()`, `capture_site()`, `build_research_pack()`, `workflow_status()`, `is_available()`.

### Oneshot CLI Harness (`oneshot_cli/` & `bin/`)
Click-based CLI application defined in `oneshot_cli/__main__.py` and exposed via binary `bin/oneshot`:
- `oneshot lanes`: Displays active routing table, lanes, and worker pools (`lanes_cmd.py`).
- `oneshot dispatch`: Dispatches task to worker via lane (`dispatch_cmd.py`, `tasks.py`). Checks git dirty state, creates isolated worktree in `../oneshot-worktrees/<task_id>`, creates task directory `.oneshot/tasks/<task_id>`, builds prompt, executes worker, writes `status.json`.
- `oneshot dispatch-many`: Executes batch dispatch across multiple tasks (`dispatch_many_cmd.py`).
- `oneshot status`: Queries status for specific task or summarizes all task states (`status_cmd.py`, `tasks.py`).
- `oneshot collect`: Collects completed worker output, merges worktree changes into working directory (`collect_cmd.py`, `tasks.py`).
- `oneshot review`: Triggers reviewer harness (`claude_code` or `codex`) on completed task diff (`review_cmd.py`, `tasks.py`).
- `oneshot escalate`: Re-dispatches failed task to fallback higher-tier lane (`escalate_cmd.py`, `tasks.py`).
- `oneshot doctor`: Runs local and SSH remote machine readiness checks (`doctor_cmd.py`). Audits python3, git, claude, opencode, gemini, codex, secrets CLI, age keys, SSH configs, worktree paths, and executes automated fixes (`--fix`).
- `oneshot memory`: Repo-first memory management (`memory_cmd.py`, `memory.py`). Manages `.oneshot/` storage, generates provenance entries (`create_provenance`), promotes decisions/blockers/runbooks, captures session summaries, indexes markdown files into global SQLite DB (`~/.local/state/oneshot/memory-index/memory.db`), and searches cross-repo abstractions.
- `oneshot worktree`: Subcommand group (`create`, `remove`, `list`) managing Git worktrees in `../oneshot-worktrees/` (`worktree.py`).

### Evaluation Framework & Operations (`eval/`, `scripts/`, `secrets/`)
- `eval/`: Benchmark sets (`eval/benchmarks/classification/`), trace outputs (`eval/traces/`), evaluation runners (`eval/scripts/`), and metric summaries (`eval/results/`).
- `scripts/`: System audit and maintenance utilities (`check-clis.sh`, `check-apis.sh`, `check-glm.sh`, `check-mcps.sh`, `check-backup.sh`, `audit-secrets.sh`, `build_instructions.py`).
- `secrets/`: SOPS/age encrypted environment files (`api.env.encrypted`, `argus.env.encrypted`, `convex.env.encrypted`, etc.).

## Canonical entry points
- **CLI Executable**: `bin/oneshot` (invokes `python3 -m oneshot_cli`).
- **Parallel Dispatch Binary**: `bin/dispatch` (wraps `python3 -m core.dispatch.run`).
- **Router CLI Resolver**: `python3 -m core.router.resolve --class <task_class> [--category <cat>]`.
- **Router Library Resolver**: `core.router.lane_policy.resolve(task_class, risk_level, category)`.
- **Parallel Dispatch Module**: `python3 -m core.dispatch.run --class <task_class> --prompt <prompt>`.
- **Argus Search API**: `core.search.argus_client.search(query, mode, max_results)`.
- **Janitor Recorder**: `core.janitor.recorder.SessionRecorder`.
- **Janitor Session Handlers**: `core.janitor.jobs.run_session_start()` & `core.janitor.jobs.run_session_end()`.
- **Memory API**: `oneshot_cli.memory` (`scaffold`, `index_repo_memory`, `search_cross_repo_abstractions`, `promote_decision`).
- **Installer Scripts**: `install.sh` and `oneshot.sh`.
- **Configuration Files**: `config/lanes.yaml`, `config/workers.yaml`, `config/models.yaml`, `config/search.yaml`, `homelab.yaml`.
