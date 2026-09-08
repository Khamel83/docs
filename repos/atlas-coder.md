# LLM-OVERVIEW — atlas-coder
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
Atlas Coder is a cost-optimized, DSPy-powered systematic programming engine and CLI assistant. It provides automated code generation, bug fixing, codebase analysis, refactoring, and project synthesis while enforcing real-time token optimization and strict cost budgets (targeting under $1/day via Gemini 2.0 Flash Lite or zero-cost local Ollama execution).

## Machine & Host Ownership
No multi-host or deployment target information is available; configuration in `homelab.yaml` lists host and runtime manager as unknown.

## What is actually built
- **`atlas_coder/` Package Core**: Modular Python package structure containing `cli` (`atlas_coder.cli.main:main`), `core`, `dspy`, `advanced`, `integrations`, and `utils`.
- **`dspy_core/` Framework Engine**: Stanford DSPy integration layer (`dspy-ai >= 2.6.27` with DSPy 3.0 upgrade support).
  - `engine.py`: Engine initialization and LLM lifecycle (`initialize_engine`, `get_engine`).
  - `model_strategy.py`: Dynamic model selection strategies matching task complexity to LLM tier.
  - `optimization.py`: Real-time cost tracking, token optimization, and budget enforcement (`get_cost_tracker`, `get_token_optimizer`).
  - `agentic.py` & `progressive_execution.py`: Multi-step agent management (`get_agentic_manager`) and progressive complexity execution (`ExecutionLevel`).
  - `bootstrap.py`: Feature bootstrapping (`bootstrap_new_feature`) and self-improvement loops (`self_improve`).
  - Primitives: `modules.py`, `cache.py`, `claude_code_parity.py`, `community.py`.
- **`generated_intelligence/` Engine Modules**: Core analytics components including `semanticanalyzer.py`, `advancedpatternrecognizer.py`, `adaptivelearner.py`, `intelligentoptimizer.py`, `rapidprocessor.py`, and `intelligenceorchestrator.py`.
- **Standalone CLI & Automation**:
  - `atlas_dspy.py` & `atlas_dspy_v6.py`: CLI interfaces supporting `bug_fix`, `generate`, `analyze`, `project`, and `refactor` workflows.
  - `watcher.py`: Background log monitoring daemon watching `~/Atlas/last_error.log` and executing `main.py` for automated traceback repair.
  - `setup_professional_structure.py` & `upgrade_dspy3.py`: Development environment scaffolding and DSPy 3.0 upgrade validation tools.
- **Dependencies & Standards**: Requires Python >= 3.11, built with `setuptools`, `click`, `rich`, `pydantic` (>=2.11.7), `python-dotenv`, and `dspy-ai`.

## Canonical entry points
- `atlas-coder`: Package CLI entry point (`atlas_coder.cli.main:main`).
- `python atlas_dspy.py` / `python atlas_dspy_v6.py`: Interactive CLI interfaces for systematic programming workflows.
- `python watcher.py`: Headless error file monitor piping tracebacks to `main.py`.
- `python upgrade_dspy3.py`: DSPy 3.0 migration and compatibility validator.
- `python3 -m pytest`: Test suite execution (`tests/unit`, `tests/integration`).
