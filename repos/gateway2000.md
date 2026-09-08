# LLM-OVERVIEW — gateway2000
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
`gateway2000` is the core routing gateway and execution abstraction layer for Oh My Pi (OMP) sessions and model providers. It provides unified CLI entry points (`g2k`, `g2k-bg`, `g2k-sensitive`, `g2k-check`, `g2k-run`) and frontdoor web interfaces connecting to `https://gateway.khamel.com`. The repository manages multi-provider task classification, provider fallback (Claude, GPT, Gemini, OpenCode Go), durable job execution, process visibility fencing, and wall-clock execution boundaries. It enforces strict cross-agent CLI invocation contracts (`agy`) and maintains systematic security, topology, and health check audit protocols.

Key architectural responsibilities include:
- Request classification and automatic routing across configured AI model backends.
- Enforcing sandbox boundaries for background jobs, git operations, and prompt execution.
- Maintaining durable worker boundaries, wall-clock timeouts, and fail-closed endpoint security.
- Providing isolated CLI abstractions for interactive, background, and sensitive operations.
- Operating standard audit, topology map, and health runbook verification flows.

## Machine & Host Ownership
No multi-host specification or host topology information is available from AGENTS.md or the current status probe output.

## What is actually built
- **Gateway Routing & Execution Wrappers (`g2k`)**:
  - Automated OMP session startup and authentication targeting `https://gateway.khamel.com`.
  - Automatic task classification with intelligent provider selection and cross-provider fallback.
  - Specialized execution lanes for standard tasks (`g2k`), low-cost background runs (`g2k-bg`), and user-authorized sensitive workloads on restricted trusted provider paths (`g2k-sensitive`).
  - Isolated client readiness verification tool (`g2k-check`) and detached execution runner (`g2k-run`).
- **Frontdoor Web Interface**:
  - Browser-accessible frontdoor entrypoint with integrated help page and specified acceptance contract (`df5938d`, `6f53ff1`, `52d89f1`, `264e0cc`).
- **Process Fencing & Durable Runtime Security**:
  - OMP process visibility isolation and environment fencing (`d4c321b`, `1d7725a`).
  - Isolated git operations and prompt execution sandbox (`642f922`).
  - Durable worker wall-clock boundary enforcement (`c0f8b52`).
  - Capability-isolated durable OMP routes and secure job route execution (`dedc9cd`, `a51e848`).
  - Fail-closed handling on durable gateway endpoints (`4ec60b1`).
  - Runtime health matching and full tunnel permission verification (`0f50ae3`).
- **Model Catalog & Recovery Infrastructure**:
  - OpenCode Go model catalog count management (`2d84a28`).
  - Temporary OpenCode Go recovery model fallback path (`243b372`).
- **Antigravity CLI Contract (`agy`)**:
  - Strict flag syntax rules requiring explicit prompt flags (`-p` / `--print`).
  - Mandatory units on timeout parameters (e.g. `600s`, `15m`; bare integers fail closed).
  - Target-gated effort flags (`--effort low|medium|high` restricted to supported `gemini-*` models).
  - Explicit mode separation between read-only audits (`--mode plan`) and interactive file edits (`--mode code`).
  - Artifact redirect conventions for audit trails.
- **Audit & Governance Infrastructure**:
  - Standardized health check runbook (`.audit/HEALTH_CHECK_RUNBOOK.md`).
  - Active audit report tracking open findings and punch lists (`.audit/AUDIT_REPORT.md`).
  - Permanent topology contract map (`.audit/SYSTEM_TOPOLOGY.md`).

## Canonical entry points
- `g2k -p "<prompt>"`: Primary Gateway2000 entry point; classifies task and executes across configured provider pool.
- `g2k-bg -p "<prompt>"`: Background lane entry point for low-cost or async tasks.
- `g2k-sensitive -p "<prompt>"`: Restricted trusted-provider lane entry point for sensitive workloads.
- `g2k-check`: Client readiness diagnostic for checking local Gateway2000 environment.
- `g2k-run <job.md>`: Detached Homelab job execution trigger.
- `agy -p "<prompt>" --mode plan|code --print-timeout <duration>`: Antigravity CLI invocation contract.
- `https://gateway.khamel.com`: Production HTTPS API gateway endpoint for OMP authentication and routing.
- Frontdoor Web Interface: Web-based browser entry point and `/help` page for gateway interaction.
- `.audit/HEALTH_CHECK_RUNBOOK.md`: Canonical runbook for repository audit, health gates, and release validation.
- `.audit/AUDIT_REPORT.md`: Live audit punch list and finding tracker.
- `.audit/SYSTEM_TOPOLOGY.md`: Permanent topology specification mapping networks, boundaries, and authority.
