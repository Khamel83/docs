# LLM-OVERVIEW — agent-os
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
Agent OS — Khamel83 Edition is an installation/distribution repository for spec-driven AI development workflows. It combines Markdown-based commands, reusable instructions, coding standards, agent definitions, setup scripts, and an optional Python integration package for Ralex. The repository describes a three-phase methodology—planning, implementation, review—with task decomposition and pattern reuse.

This is primarily an agent-configuration and methodology toolkit, not an application server. No `AGENTS.md` is currently present despite the authority notice above; agents must therefore derive repository mechanics from `README.md`, command definitions, instruction files, standards, and implementation code without treating this overview as policy.

Operational evidence is limited: the live status probe was disabled. Recent commits establish active repository maintenance and add a Homelab contract, but do not prove a running production deployment.

## Machine & Host Ownership
No concrete machine or multi-host deployment information is available. `homelab.yaml` assigns ownership to `homelab`, declares production host and manager as `unknown`, names the intended unit `agent-os`, and places monitoring in `standby`.

## What is actually built
- **Installation layer**
  - `install.sh`: documented one-command installation target fetched from the repository’s `main` branch.
  - `setup.sh`: general setup entry point.
  - `setup-claude-code.sh`: Claude Code-specific setup.
  - `setup-cursor.sh`: Cursor-specific setup.
  - `install-khamel83.sh`: Khamel83-edition installation path.
  - README claims configuration support for Claude Code, Cursor, Gemini CLI, and Ralex; only the setup/install paths visible in the supplied tree should be assumed implemented.

- **Agent and workflow definitions**
  - `claude-code/agents/`: Claude Code subagent definitions. Commit history identifies context-fetcher, test-runner, git-workflow, file-creator, and date-checker agents, but their current file-level contracts were not included in the observed source.
  - `commands/plan-product.md`: product-planning workflow definition.
  - `commands/analyze-product.md`: existing-product analysis workflow.
  - `commands/create-spec.md`: specification-creation workflow; commit history records date-checker integration.
  - `commands/execute-tasks.md`: task-execution workflow.
  - `commands/khamel83-integration.md`: Khamel83-specific integration workflow.
  - `instructions/core/` and `instructions/meta/`: reusable instruction sets; exact files and semantics were not observed.
  - `standards/best-practices.md`, `standards/code-style.md`, `standards/code-style/`, and `standards/tech-stack.md`: project guidance distributed to agent environments. Their contents were not observed, so no concrete language or framework rules can be inferred.
  - Root `templates/` is empty in the supplied tree.

- **Ralex integration package**
  - `ralex-integration-package/agent_os_bridge.py`
    - Defines `TaskRequest` with `description`, optional `context`, and optional `preferences`.
    - Defines `TaskPlan` with a methodology identifier, phase list, complexity classification, available-pattern names, and per-purpose model recommendations.
    - `AgentOSBridge` is the central integration façade and imports `AgentOSContextAnalyzer`, `MethodologyEngine`, and `PatternManager` as sibling modules.
    - The observed source does not establish an HTTP, RPC, or CLI transport; callers use Python objects directly.
  - `ralex-integration-package/context_analyzer.py`
    - Defines `AgentOSContextAnalyzer(project_root=".")`.
    - Uses `pathlib`, filesystem access, JSON, YAML, subprocess execution, and timestamps.
    - `get_full_context()` assembles project-root metadata, an ISO timestamp, Agent OS structure analysis, Git context, project state, the current specification, and subsequent task-related context.
    - Internal methods named in the observed code include `_analyze_agent_os_structure`, `_analyze_git_context`, `_analyze_project_state`, and `_find_current_spec`.
  - `ralex-integration-package/methodology_engine.py`
    - Defines the `TaskBreakdown` dataclass: original task, selected methodology, phases, micro-tasks, estimated time, and success criteria.
    - Defines `MethodologyEngine`, optionally parameterized by a templates path.
    - `apply_three_phase_methodology(task_description, context=None)` is the observed planning API. Its output models planning, implementation, and review as explicit phases plus micro-tasks.
  - `ralex-integration-package/pattern_manager.py`
    - Defines the `Pattern` dataclass with identity, description, task type, complexity, components, success rate, usage count, last-used timestamp, template payload, tags, and created/updated dates.
    - Defines `PatternManager(project_root=".")`, which initializes an in-memory cache and loads persisted patterns through `_load_patterns()`.
    - Exposes matching behavior through `find_matching_patterns(...)`; the observed signature includes a configurable threshold, but the full implementation and persistence location were not supplied.
    - Uses JSON serialization, filesystem paths, dataclass conversion, timestamps, hashing, and regular expressions.
  - `ralex-integration-package/integration_examples.py`
    - Defines `RalexAgentOSIntegration`.
    - Instantiates `AgentOSBridge` and records whether the working directory is recognized as an Agent OS project.
    - `handle_user_request(user_input)` is asynchronous and dispatches lifecycle phrases such as `start-project`/`start project` and `resume-project`/`resume project` before general request handling.
    - This file is an integration example, not evidence of a deployed daemon.
  - `ralex-integration-package/agent_os_config.yaml`: Ralex/Agent OS integration configuration; contents were not observed.
  - `ralex-integration-package/agent_os_templates/`: integration-specific templates.
  - `ralex-integration-package/INTEGRATION_PLAN.md`: design/planning documentation, not runtime proof.

- **Runtime and operational configuration**
  - `homelab.yaml` uses schema `homelab.project/v1`.
  - Project identity: `agent-os`, repository `https://github.com/Khamel83/agent-os`, lifecycle `development`, owner `homelab`.
  - It classifies the project as `kind: service`, but no service process, web framework, listener, container definition, or executable daemon is present in the observed source.
  - Production metadata names unit `agent-os`; host and manager remain unknown.
  - Monitoring is `standby`; health checks are empty.
  - Observability points to `homelab-api:/observations` with sanitized evidence.
  - Required and optional dependency lists are empty.
  - Secret references and repair IDs are empty; `broker_consumer` is `agent-os`.
  - Documentation configuration names `README.md` as the index and `docs/OPERATIONS.md` as the operations guide. `docs/OPERATIONS.md` does not appear in the supplied directory tree and must not be assumed present.

- **Dependencies and data boundaries**
  - The Python integration directly imports PyYAML via `yaml`; remaining observed imports are Python standard-library modules.
  - Modules communicate through Python dataclasses, dictionaries, lists, filesystem state, YAML/JSON documents, and Git/subprocess-derived context.
  - No package manifest, lockfile, Python packaging metadata, database engine, migration set, database tables, message broker client, web framework, API routes, authentication layer, or network listener was observed.
  - No external service dependency is declared in `homelab.yaml`. The observability sink is configuration metadata only; supplied code does not prove an implemented sender.
  - Persistent pattern storage is implied by `PatternManager._load_patterns()`, but its exact file path, schema, mutation behavior, and concurrency guarantees are not visible in the supplied excerpt.

## Canonical entry points
- **Install for end users:** `install.sh`, documented as `curl -sSL https://raw.githubusercontent.com/Khamel83/agent-os/main/install.sh | bash`.
- **Configure supported agent environments:** `setup.sh`, `setup-claude-code.sh`, `setup-cursor.sh`, and `install-khamel83.sh`.
- **Run the documented development lifecycle:** begin with `commands/plan-product.md` for a new product or `commands/analyze-product.md` for an existing codebase; create work specifications through `commands/create-spec.md`; execute them through `commands/execute-tasks.md`.
- **Use Khamel83-specific workflow integration:** `commands/khamel83-integration.md`.
- **Embed from Python/Ralex:** construct `AgentOSBridge` from `ralex-integration-package/agent_os_bridge.py`; pass a `TaskRequest`; use `AgentOSContextAnalyzer` for repository context, `MethodologyEngine` for task decomposition, and `PatternManager` for reusable-pattern lookup.
- **Study asynchronous Ralex dispatch:** `RalexAgentOSIntegration.handle_user_request()` in `ralex-integration-package/integration_examples.py`.
- **Configure Ralex integration:** `ralex-integration-package/agent_os_config.yaml` and `ralex-integration-package/agent_os_templates/`.
- **Repository orientation:** `README.md`; release history in `CHANGELOG.md`; edition-specific changes in `KHAMEL83_ENHANCEMENTS.md`.
- **Operational registration:** `homelab.yaml`. Treat it as a development/standby contract, not proof that a production unit is installed or running.
- **Validation boundary:** no live status evidence is available. Before asserting runtime health, deployment state, supported setup behavior, or successful integration, inspect the relevant scripts/configuration and execute the applicable installation or Python entry path in a controlled environment.
