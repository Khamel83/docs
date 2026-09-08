# LLM-OVERVIEW — HR_AI
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is

A flat Python application for local analysis of USC human-resources Excel snapshots. Its implemented center is `USCHRAnalytics` in `usc_hr_analytics_v3.py`, backed by DuckDB and pandas, with configuration in `config.py`, database setup/encryption utilities, a Flask-oriented web launcher, and a single dashboard template.

The repository describes natural-language querying, Ollama integration, encrypted storage, historical snapshots, and dynamically generated questions. Treat those as repository claims rather than fully verified runtime capabilities: the supplied status probe is disabled, no running deployment is evidenced, and the visible source tree lacks `usc_hr_local_llm.py` even though `test_messy_questions.py` imports it.

No `AGENTS.md` is currently present. Until one is added, derive behavior from source and current repository-local documentation without treating this overview as policy.

## Machine & Host Ownership

No multi-host deployment information is available. `config.py` binds the web interface to loopback at `127.0.0.1:5000`; `homelab.yaml` records production host and manager as `unknown`, lifecycle `development`, and monitoring state `standby`. The status probe was not run, so no active machine or service instance is established.

## What is actually built

### Runtime shape

- Python 3.9+ application organized as top-level modules rather than an installable package.
- Local filesystem data root: `data/`.
- Expected source spreadsheets: `data/excel_samples/`.
- Expected generated databases and logs: `data/processed_data/`.
- Expected runtime logs directory: `data/logs/`.
- Primary libraries evidenced by imports and startup checks:
  - DuckDB for local analytical storage and SQL execution.
  - pandas for tabular data handling.
  - openpyxl for Excel input support.
  - Flask for the web surface.
  - python-dotenv for `.env` loading.
  - requests for local model HTTP interaction in model tests.
- `requirements.txt` is the dependency manifest; exact pinned versions were not supplied.

### Configuration

`config.py` defines the central `Config` class:

- `BASE_DIR`: repository directory containing `config.py`.
- `DATA_DIR`: `<repo>/data`.
- `EXCEL_SAMPLES_DIR`: `<repo>/data/excel_samples`.
- `PROCESSED_DATA_DIR`: `<repo>/data/processed_data`.
- `LOGS_DIR`: `<repo>/data/logs`.
- `DATABASE_PATH`: `data/processed_data/usc_hr_analytics.db`.
- `ENCRYPTED_DATABASE_PATH`: `data/processed_data/usc_hr_analytics_encrypted.db`.
- `DATABASE_LOG_PATH`: `data/processed_data/usc_hr_analytics.log`.
- `USE_ENCRYPTION`: defaults to `False` and is intended to be overridden from environment configuration.
- `DB_PASSWORD`: defaults to `None` and is intended to be loaded from `.env`.
- Web defaults: host `127.0.0.1`, port `5000`, debug disabled.

`load_config.py` exposes `get_db_connection`, imported by the main analytics engine. Its connection-selection and encryption behavior were not included in the supplied source excerpts; inspect it before changing database-open semantics.

### Core analytics and ingestion

`usc_hr_analytics_v3.py` contains `USCHRAnalytics`, the main application class.

Construction:

1. Resolves input directory from an explicit `data_dir` or `Config.EXCEL_SAMPLES_DIR`.
2. Resolves database path from an explicit `db_path` or `Config.DATABASE_PATH`.
3. Initializes `self.conn`.
4. Calls `setup_logging()`.
5. Calls `setup_database()`.
6. Defines filename-pattern mappings for dated Excel exports.

Observed input categories include:

- `emp_roster`, matched by `EmpRoster_<six digits>.xls` or `.xlsx`.
- `emp_comp`, matched by `EmpComp_<six digits>.xls` or `.xlsx`.
- `race_ethnicity`, with a similarly dated filename pattern whose full expression was not supplied.

The engine imports:

- pandas and DuckDB for ingestion/query processing.
- `datetime` and `date` for snapshot/date handling.
- `pathlib.Path` for filesystem paths.
- `logging` for runtime diagnostics.
- `re` for export filename classification.
- `Config` and `get_db_connection` for centralized paths and database access.

The supplied excerpts do not expose the complete table schema, SQL views, ingestion methods, query methods, or cleanup lifecycle. Do not invent table or column names; inspect `usc_hr_analytics_v3.py`, `data_access.py`, and the live DuckDB schema before modifying queries.

### Database access and encryption

Database-related modules present:

- `load_config.py`: provides the connection helper consumed by `USCHRAnalytics`.
- `data_access.py`: dedicated data-access module; implementation was not supplied.
- `database_encryption.py`: encryption support module; implementation was not supplied.
- `create_encrypted_db.py`: standalone conversion/setup script.
- `setup_encrypted_db.py`: encrypted-database setup entry point.
- `simple_encrypt.py`: additional encryption utility.
- `config.py`: owns plaintext/encrypted database paths and encryption defaults.

`create_encrypted_db.py` performs the following observed work:

- Calls `dotenv.load_dotenv()`.
- Reads the secret from environment variable `DuckDB_PW`.
- Raises `ValueError` when `DuckDB_PW` is absent.
- Uses `data/processed_data/usc_hr_analytics.db` as the source path.
- Uses `data/processed_data/usc_hr_analytics_encrypted.db` as the destination path.
- Returns `False` after password-loading failures.
- Logs only password length in the visible code, not the password value.

The README’s “AES-256” and “military-grade encryption” wording is not substantiated by the supplied implementation excerpt. Verify the actual DuckDB encryption mechanism and key handling in `database_encryption.py`, `setup_encrypted_db.py`, and `load_config.py` before repeating that guarantee or changing storage behavior. Plaintext storage remains the configured default because `Config.USE_ENCRYPTION` starts as `False`.

Secrets must remain external to source control. The observed secret contract is `.env` → `DuckDB_PW`; no other secret references are declared by `homelab.yaml`.

### Schema abstraction

`schema_abstraction.py` defines `USCHRSchemaAbstraction`.

Construction eagerly:

1. Instantiates `USCHRAnalytics`, which can initialize logging and database state.
2. Calls `build_schema_abstraction()`.
3. Stores the resulting dictionary as `self.schema`.

The abstraction’s visible metadata describes:

- Database name: USC HR Analytics Database.
- Content: employee, demographic, compensation, and operational HR data.
- Update-frequency claim: weekly.
- Retention claim: historical snapshots from 2022 onward.

The module imports JSON and pandas and returns typed dictionaries/lists. It is intended to expose schema metadata rather than employee records to an LLM, but the supplied excerpt does not establish what fields, samples, statistics, or values are included. Treat the abstraction as sensitive until the full generated payload is inspected. Instantiating it is not metadata-only from a runtime perspective because it also constructs the analytics engine.

### Natural-language and local-model integration

Repository documentation advertises local Ollama-backed natural-language-to-SQL behavior. Evidence in supplied source is limited:

- `test_messy_questions.py` compares model names such as `llama3.2:3b` and `gemma3:latest`.
- It constructs `USCHRAnalytics`.
- It imports and constructs `USCHRLocalLLM`.
- It assigns `llm.model`.
- It calls `llm.execute_question(question)` and expects `(response, dataframe)`.
- It records execution time.
- It imports `requests` and JSON, consistent with an HTTP model endpoint.

However, `usc_hr_local_llm.py` is absent from the supplied directory tree. Therefore the model test has an unresolved local import in this checkout unless that implementation is generated, ignored, or supplied externally. No production model endpoint, prompt construction, SQL validation, authorization boundary, or Ollama availability check is established by the excerpts.

Related design/reference documents exist:

- `NATURAL_LANGUAGE_CAPABILITY.md`
- `OLLAMA_MODEL_GUIDE.md`
- `PHASE_2_LLM_INTEGRATION_PLAN.md`
- `DATA_RESEARCH_QUESTIONS.md`

A file named as a plan is not evidence of completed functionality. Reconcile these documents against current source before treating their designs as active.

### Web interface

`start_web.py` is the observed launcher for the web surface.

Startup checks:

- Imports Flask, DuckDB, pandas, and openpyxl.
- Directs users to `pip install -r requirements.txt` when an import fails.
- Reads `Config.DATABASE_PATH`.
- Reads `Config.EXCEL_SAMPLES_DIR`.
- Rejects startup preparation when the configured database does not exist.
- Opens the database with `duckdb.connect(..., read_only=True)` to inspect it.

The visible startup code does not contain Flask route definitions. `templates/dashboard.html` is the only observed template. No static directory, API blueprint, authentication layer, session store, CSRF protection, or route map is established by the supplied tree. Locate the actual Flask application and route handlers before changing browser/API behavior; they may reside later in `start_web.py` or another top-level module.

The configured server is local-only by default. Changing `WEB_HOST` to a non-loopback address changes the trust boundary and requires reviewing data exposure, debug mode, query validation, and database credentials.

### Quick-question generation

`generate_quick_questions.py` is a standalone generator associated with the README’s quick-question feature. The implementation was not supplied, so the claimed count of 62 and the claim that questions are derived dynamically from data patterns are not independently verified here.

### Error handling

`error_handling.py` is the repository’s dedicated error-handling module. Its exception types, decorators, logging policy, and caller coverage were not supplied. Reuse its existing conventions rather than adding a second error representation.

### Demonstration and validation surfaces

Observed executable or validation-oriented files:

- `demo.py`: demonstration entry point.
- `test_system.py`: system-level test script.
- `comprehensive_test_suite.py`: broader test runner or test collection.
- `test_messy_questions.py`: manual/model comparison harness.
- `generate_quick_questions.py`: generation utility.
- `create_encrypted_db.py`: plaintext-to-encrypted database utility.
- `setup_encrypted_db.py`: encrypted storage setup.
- `start_web.py`: web launcher.

`test_messy_questions.py` mutates the selected model on a `USCHRLocalLLM` instance and exercises a real question path rather than an isolated unit. It depends on the missing local-LLM module, database availability, model availability, and likely a running Ollama service. It should not be assumed hermetic.

No CI configuration, packaging metadata, container definition, migration framework, or conventional `tests/` package appears in the supplied tree.

### Operations and deployment metadata

`homelab.yaml` declares:

- Contract schema: `homelab.project/v1`.
- Project ID: `hr-ai`.
- Repository: `https://github.com/Khamel83/HR_AI`.
- Kind: `service`.
- Lifecycle: `development`.
- Owner: `homelab`.
- Monitoring state: `standby`.
- Production unit name: `hr-ai`.
- Production host: unknown.
- Runtime manager: unknown.
- Health source: `homelab-api`.
- No health checks.
- Observation sink: `homelab-api:/observations`.
- Evidence policy: sanitized.
- No declared required or optional dependencies.
- Secret broker consumer: `hr-ai`.
- No declared secret references.
- No repair IDs.
- Documentation index: `README.md`.
- Operations-document path: `docs/OPERATIONS.md`.

The supplied tree contains no `docs/` directory, so the declared operations-document path is currently unresolved. The homelab contract does not prove deployment or health; it explicitly records standby monitoring and unknown production ownership.

### Documentation inventory and reliability

Operational/user documentation present:

- `README.md`
- `HOW_TO_START.md`
- `QUICK_START.md`
- `DEPLOYMENT_CHECKLIST.md`
- `FINAL_DEPLOYMENT_SUMMARY.md`
- `PROJECT_STRUCTURE.md`

Encryption/security documentation present:

- `ENCRYPTION_GUIDE.md`
- `QUICK_ENCRYPTION.md`
- `SECURITY_GUARANTEE.md`

Architecture/history documentation present:

- `USC_HR_ANALYTICS_PLAN.md`
- `PHASE_2_LLM_INTEGRATION_PLAN.md`
- `REFACTORING_ASSESSMENT.md`
- `REFACTORING_SUMMARY.md`
- `CLEAN_PROJECT_SUMMARY.md`
- `CLAUDE.md`

Treat summaries, guarantees, plans, and deployment checklists as historical or aspirational until matched to source and runtime evidence. The latest visible commit adds the homelab contract; the disabled status probe provides no evidence that the application is currently running.

## Canonical entry points

### Run the web surface

`python start_web.py`

This is the user-facing launcher evidenced by source. It validates core Python imports and the configured DuckDB file before proceeding. Expected default address is `http://127.0.0.1:5000`. Exact route availability is not established by the supplied excerpts.

### Use the analytics engine programmatically

```python
from usc_hr_analytics_v3 import USCHRAnalytics

analytics = USCHRAnalytics()
```

Default construction uses `data/excel_samples` and `data/processed_data/usc_hr_analytics.db`, initializes logging, and sets up the database. Pass `data_dir` or `db_path` explicitly for isolated tooling or tests.

### Access the database connection policy

```python
from load_config import get_db_connection
```

Use this existing helper rather than opening a second connection path when application encryption/configuration semantics must be preserved. Direct read-only inspection is already used by `start_web.py`.

### Build schema metadata

```python
from schema_abstraction import USCHRSchemaAbstraction

schema = USCHRSchemaAbstraction().schema
```

This also instantiates `USCHRAnalytics`; it may touch database and logging state. Audit the resulting payload before transmitting it outside the machine.

### Configure runtime paths and web defaults

```python
from config import Config
```

Change central paths and server settings here rather than embedding additional path constants. Preserve `pathlib`-based repository-relative behavior.

### Prepare encrypted storage

`python create_encrypted_db.py`

Requires `DuckDB_PW` in `.env`. Reads the configured plaintext database location conceptually, although this script’s visible path literals duplicate the same paths rather than referencing `Config`.

`python setup_encrypted_db.py`

Additional setup entry point; inspect its behavior and idempotency before use.

### Run available validation scripts

- `python test_system.py`
- `python comprehensive_test_suite.py`
- `python test_messy_questions.py`

The model comparison script is not currently self-contained because `usc_hr_local_llm.py` is absent from the observed repository tree. It also requires model and database runtime dependencies.

### Supporting executables

- `python demo.py` — demonstration flow; inspect before relying on it as the canonical production startup.
- `python generate_quick_questions.py` — quick-question generation.
- `python simple_encrypt.py` — alternate encryption utility; determine its relationship to the canonical encrypted database workflow before use.

### Repository maintenance

- `git status --short`
- `git log --oneline -5`

Shell scripts `push_commands.sh` and `push_to_github.sh` are repository publishing helpers, not application runtime entry points. Inspect them before execution because their git and remote side effects are not described by the supplied source.
