# LLM-OVERVIEW — contextpack
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is

`contextpack` is a Python 3.11+ command-line package that scans a Git repository and generates an AI-agent handoff packet describing the repository, detected technology, commands, instructions, risks, and likely next steps. Package metadata declares version `0.1.0`; the console command is `contextpack`.

The implemented flow is:

1. Accept a repository path through the Typer CLI.
2. Resolve and inspect that repository in `contextpack.scanner`.
3. Detect languages, frameworks, commands, files, agent guidance, Git metadata, and possible secret-bearing files.
4. Represent the scan through Pydantic-backed models.
5. Render Markdown and JSON artifacts through the generator/template layer.
6. Write a packet, normally under the scanned repository’s `.contextpack/` directory.
7. Support repository aggregation through the later bundle functionality and the committed `ALL-REPOS.md` inventory.

No `AGENTS.md` is present in the observed tree. The header’s authority reference is therefore prospective rather than evidence of current repository-local agent instructions.

Repository state is development/standby, not demonstrated production service operation. `homelab.yaml` classifies it as a `service`, lifecycle `development`, monitoring state `standby`, with no configured health checks, external dependencies, secret references, or repair actions.

## Machine & Host Ownership

No multi-host information is available: the live status probe was disabled, and `homelab.yaml` records the production host and manager as `unknown`.

## What is actually built

### Package layout and responsibilities

- `contextpack/__init__.py` — package initializer. No exported API or initialization behavior is established by the supplied source excerpt.
- `contextpack/cli.py` — Typer application module. `pyproject.toml` exposes its `app` object as the installed `contextpack` command via `contextpack.cli:app`. Recent commit history records a subsequently added bundle command, but the supplied `cli.py` body is not available, so its exact options and dispatch paths are not established here.
- `contextpack/scanner.py` — repository orchestration and filesystem/Git inspection.
- `contextpack/detectors.py` — technology and command detection. `scanner.py` imports `detect_all` and `NOISE_DIRS`, making this the centralized detection layer and directory-exclusion vocabulary.
- `contextpack/models.py` — structured scan and risk data. The observed scanner imports `RepoScan` and `RiskItem`; Pydantic 2 is the declared modeling dependency.
- `contextpack/safety.py` — safety-sensitive file inspection. `scanner.py` imports `detect_secret_files`, indicating that possible secret files are identified during scanning rather than emitted indiscriminately.
- `contextpack/generators.py` — transforms a `RepoScan` into packet files. Observed public functions include `write_all`, `gen_repo_summary`, `gen_commands`, and `gen_risks`.
- `contextpack/templates.py` — shared rendering text/templates. Exact template constants and formatting rules are not present in the supplied excerpts.

The architecture is a direct pipeline rather than a server or plugin framework:

`CLI → scan_repo(path) → detectors/safety/Git probes → RepoScan/RiskItem models → generators/templates → Markdown and JSON files`

There are no observed HTTP routes, database tables, queues, background workers, network clients, or persistent stores in this repository. The FastAPI route shown in `tests/fixtures/python_fastapi/main.py` is fixture input used to test detection; it is not a `contextpack` service endpoint.

### Scanner subsystem

`scan_repo(path: Path) -> RepoScan` is the central programmatic operation.

Observed behavior and inputs:

- Expands `~` and resolves the supplied path to an absolute path before scanning.
- Rejects a missing path; `tests/test_scanner.py` expects `SystemExit` for a known nonexistent directory.
- Uses `detect_all` for consolidated repository detection.
- Uses `detect_secret_files` for safety/risk discovery.
- Uses a private `_git(cmd, cwd)` helper for Git metadata.
- `_git` executes `git` with the supplied argument list through `subprocess.run`.
- Git calls capture stdout/stderr, decode text, and have a five-second timeout.
- Git errors, timeouts, and invocation failures degrade to an empty string rather than propagating an exception.
- The supplied scanner excerpt ends during `scan_repo`; exact traversal depth, generated risk rules, and every populated `RepoScan` field are therefore not established.

Scanner-controlled vocabularies observed in source:

- Agent instruction filenames:
  - `CLAUDE.md`
  - `AGENTS.md`
  - `.cursorrules`
  - `AGENT.md`
  - `AI.md`
  - `CLAUDE.local.md`
- Documentation filenames:
  - `README.md`
  - `README.rst`
  - `README.txt`
  - `CHANGELOG.md`
  - `CONTRIBUTING.md`
- Candidate entry-point filenames:
  - Python: `main.py`, `app.py`, `cli.py`, `server.py`, `run.py`
  - JavaScript/TypeScript: `index.js`, `index.ts`, `server.js`, `server.ts`
  - Go: `main.go`
  - Rust: `main.rs`
  - Java: `main.java`
- Test/config indicators:
  - `pytest.ini`
  - `jest.config.js`
  - `jest.config.ts`
  - `vitest.config.ts`
  - `go.sum`

These are filename heuristics, not proof that an identified file is executable or that a framework is operational.

### Detection subsystem

`contextpack.detectors.detect_all` supplies the scanner’s main detection pass. The tests establish observable detection outcomes:

- A Python/FastAPI fixture is reported with `Python` in `scan.languages`.
- A fixture containing `CLAUDE.md` sets `scan.has_agent_instructions`.
- The Python fixture yields at least one install command and at least one test command.
- A Node/Next.js fixture reports `Next.js` in `scan.frameworks`.
- The Next.js fixture yields development, test, and build commands; its development commands include `npm run dev` or another command containing `dev`.
- A shell fixture reports `Shell` and preserves the repository directory name as `scan.name`.

`NOISE_DIRS` is imported by the scanner, so traversal excludes or specially handles a shared detector-defined set of irrelevant/generated directories. The exact exclusion list is not shown and should be read from `detectors.py` before changing scan coverage.

Detected commands are grouped structurally. At minimum, observed consumers use:

- `scan.commands.install`
- `scan.commands.dev`
- `scan.commands.test`
- `scan.commands.build`

Do not replace these lists with a single command string without migrating scanners, generators, models, and tests.

### Model subsystem

`RepoScan` is the aggregate scanner result consumed by generators and tests. Fields demonstrated by source and tests include:

- `name`
- `languages`
- `frameworks`
- `has_agent_instructions`
- `commands`
  - `install`
  - `dev`
  - `test`
  - `build`

`RiskItem` is part of scanner/generator risk handling, but its fields and severity vocabulary are not visible in the supplied excerpts.

Pydantic `>=2.0` is the only declared data-modeling dependency. Any changes to serialized field names affect both `contextpack.json` and Markdown generation and must be treated as an output-contract change.

### Generation subsystem

`write_all(scan, target=None)` is the tested packet-writing API.

For a scanned repository and default target, it returns paths for exactly these seven generated artifacts:

- `repo_summary.md`
- `file_map.md`
- `commands.md`
- `agent_instructions.md`
- `risks.md`
- `next_prompt.md`
- `contextpack.json`

The tests require every returned path to exist and contain nonzero bytes. Default output is described by the test comment as `.contextpack/` within the scanned repository.

The artifact roles implied by names and public generator functions are:

- `repo_summary.md` — compact repository identity and detected stack.
- `file_map.md` — important-file inventory derived from scanning.
- `commands.md` — detected install, development, build, and test commands.
- `agent_instructions.md` — collected/indexed repository-local AI instruction files.
- `risks.md` — scan risks, including safety-sensitive findings.
- `next_prompt.md` — generated continuation prompt for an incoming agent.
- `contextpack.json` — machine-readable serialization of the packet/scan.

Observed specialized generator functions:

- `gen_repo_summary`
- `gen_commands`
- `gen_risks`

`write_all` is intended to be repeatable: `test_write_all_idempotent` invokes it twice against the same copied fixture. The supplied test excerpt ends before its final assertions, so the exact idempotence contract—stable content, no duplicates, or merely no exception—is not visible.

Generators overwrite or refresh generated outputs; callers should treat `.contextpack/` as derived state rather than authored source.

### Safety subsystem

`detect_secret_files` is invoked from scanning. This establishes secret-file detection, not secret-value extraction or secret management.

The Homelab contract reinforces the safety boundary:

- `observability.evidence_policy: sanitized`
- `secrets.broker_consumer: contextpack`
- `secrets.references: []`

No actual broker integration, credential references, or secret backend dependency appears in `pyproject.toml`. Do not infer that secrets are fetched or validated. The observed implementation only establishes detection of files that may contain secrets.

### Bundle and repository inventory

Recent history records:

- Initial implementation.
- Addition of a bundle command and `ALL-REPOS.md` covering 47 repositories.
- Addition of the Homelab project contract.

`ALL-REPOS.md` is therefore a generated or maintained aggregate inventory associated with bundle functionality. Its schema and regeneration command are not included in the supplied excerpts; inspect `contextpack/cli.py` and generator code before editing it manually.

### Runtime and external dependencies

Declared runtime:

- Python `>=3.11`
- `typer>=0.9` — CLI declaration, argument parsing, and command dispatch.
- `rich>=13.0` — terminal rendering.
- `pydantic>=2.0` — structured scan/risk models and JSON-compatible data.

Build system:

- `setuptools>=68`
- Backend: `setuptools.build_meta`

External executable dependency:

- `git`, invoked by `scanner._git`. Git probe failures are tolerated and represented by empty output.

No application database, web server, container definition, daemon configuration, or deployment manifest is present in the observed tree.

### Homelab contract

`homelab.yaml` declares:

- Schema: `homelab.project/v1`
- Project ID: `contextpack`
- Name: `Contextpack`
- Repository: `https://github.com/Khamel83/contextpack`
- Kind: `service`
- Lifecycle: `development`
- Owner: `homelab`
- Monitoring state: `standby`
- Runtime unit: `contextpack`
- Production host: `unknown`
- Runtime manager: `unknown`
- Health source: `homelab-api`
- Health checks: none
- Observation sink: `homelab-api:/observations`
- Evidence policy: `sanitized`
- Required dependencies: none
- Optional dependencies: none
- Secret references: none
- Repair IDs: none

The contract names `README.md` as the documentation index and `docs/OPERATIONS.md` as operations documentation, but neither path appears in the supplied directory tree. Treat those references as unresolved configuration, not proof the documents exist.

### Tests and fixtures

Pytest discovers tests under `tests/`.

Test organization:

- `tests/conftest.py` provides paths to fixture repositories.
- `tests/test_scanner.py` covers detection and invalid input.
- `tests/test_generators.py` covers packet creation and repeated generation.
- `tests/fixtures/python_fastapi` represents a Python/FastAPI repository.
- `tests/fixtures/node_next` represents a Node/Next.js repository.
- `tests/fixtures/simple_bash` represents a shell repository.

Fixture code is scanned as data. In particular, the FastAPI application and its `GET /` route returning `{"status": "ok"}` do not belong to the shipped `contextpack` runtime.

The source/test excerpts in the repository content are truncated mid-definition. They are sufficient to establish the interfaces above but not sufficient to conclude that the checked-in files themselves end at those points.

## Canonical entry points

### Installed CLI

```text
contextpack
```

Defined by:

```toml
[project.scripts]
contextpack = "contextpack.cli:app"
```

`contextpack.cli:app` is the canonical user-facing command dispatcher. Use its existing Typer commands and options rather than constructing a second executable wrapper. The recent bundle feature should also be reached through this application.

### Programmatic scan API

```python
from pathlib import Path
from contextpack.scanner import scan_repo

scan = scan_repo(Path("/path/to/repository"))
```

`scan_repo` is the canonical boundary from repository filesystem state to a `RepoScan`. It owns path normalization, detector orchestration, Git probing, agent/document/entry-point discovery, and safety findings.

### Programmatic generation API

```python
from contextpack.generators import write_all

written_paths = write_all(scan, target=None)
```

`write_all` is the canonical complete-packet writer. Prefer it when all standard outputs are required. Use specialized generators such as `gen_repo_summary`, `gen_commands`, and `gen_risks` only when an individual rendered section is explicitly needed.

### Canonical configuration and metadata

- `pyproject.toml` — package metadata, Python floor, runtime dependencies, console script, build backend, and pytest discovery.
- `homelab.yaml` — project ownership and operational registration; currently standby with unknown production placement.
- `ALL-REPOS.md` — aggregate multi-repository output associated with bundle functionality.
- `contextpack/models.py` — authoritative in-process and serialized data contracts.
- `contextpack/detectors.py` — authoritative detection rules and noise-directory policy.
- `contextpack/templates.py` — authoritative generated prose/layout definitions.

### Canonical verification

```text
pytest
```

Pytest is configured to search `tests/`. The highest-value existing behavioral checks are:

- Python, Next.js, and shell technology detection.
- Detection of repository-local agent instructions.
- Discovery of install/dev/test/build commands.
- Rejection of a missing repository path.
- Creation of all seven standard output files.
- Repeated invocation of `write_all` against the same repository.

When changing a detector, validate against the relevant fixture and generated consumer behavior. When changing a model, update scanner population, JSON serialization, generators, and all field consumers together. When changing output filenames or packet composition, update `write_all`, `EXPECTED_FILES`, bundle behavior, and any aggregate inventory generation in one cutover.
