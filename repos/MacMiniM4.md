# LLM-OVERVIEW — MacMiniM4
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
Monorepo repository hosted on an M4 Mac Mini (`macminim4`) containing personal automation projects, utility scripts, and local LLM workflow pipelines. Maintained under the `homelab` organization with a standby service lifecycle defined by `homelab.yaml`.

## Machine & Host Ownership
No multi-host information is available; code configurations target local Mac Mini execution (`/Users/macmini/...`) with `homelab.yaml` reporting runtime host as `unknown` and unit `macminim4`.

## What is actually built
The repository consists of homelab service metadata and six discrete project directories under `MacMiniM4/`:

- **Homelab Contract (`homelab.yaml`)**:
  - Service metadata defining ID `macminim4`, `service` kind, `development` lifecycle, and `standby` monitoring state.
  - Standardized health monitoring pointers (`homelab-api`), documentation links (`README.md`, `docs/OPERATIONS.md`), and secret consumer tag (`macminim4`).

- **Instapaper Automation & LLM Synthesis (`MacMiniM4/instapaper/`)**:
  - **Authentication & Scraping**: Automated login and session handling for `https://www.instapaper.com/user/login` using `requests.Session` and `BeautifulSoup` to extract hidden CSRF/form fields (`instapaper.py`, `instapaper1.py`).
  - **Local LLM Integration**: Article analysis and summarization powered by local LLaMA models via `langchain_ollama.OllamaLLM` (`instapaper111124.py`, `instapaper122324v2.py`, `instapaper122324v3.py`).
  - **Concurrency & Configuration**: Parallel article processing via `ThreadPoolExecutor`, randomized `user_agents`, retry mechanisms (`max_retries`), and configuration loading from `/Users/macmini/Library/Mobile Documents/com~apple~CloudDocs/Code/instapaper/config.json`.
  - **Dynamic Dependency Bootstrapping**: Self-contained setup script (`instapaper111124.py`) that checks and auto-installs missing Python packages (`requests`, `pandas`, `langchain-ollama`) via `pip` at execution time.
  - **Export & Verification**: Structured output generation using `pandas` (CSV) and `python-docx` (`Document`). Includes parsing error detection helper (`quick.py`) checking for `"Sorry, there was an error parsing that article."` in cached HTML documents.

- **Additional Utility Subsystems (`MacMiniM4/`)**:
  - **`emailprocessing/`**: Email extraction and batch processing tools.
  - **`slack/`**: Slack bot and message automation scripts.
  - **`story generator/`**: Narrative and content generation tools using LLMs.
  - **`video dedupe/`**: Video file hash analysis and deduplication tools.
  - **`old_notused/`**: Archive folder for deprecated scripts and previous code iterations.

## Canonical entry points
- **Homelab Contract**: `homelab.yaml`
- **Instapaper Active Pipelines**:
  - `MacMiniM4/instapaper/instapaper122324v3.py` (Latest Ollama LLM synthesis script)
  - `MacMiniM4/instapaper/instapaper122324v2.py` (LLM processing variant)
  - `MacMiniM4/instapaper/instapaper111124.py` (Self-bootstrapping LLM script)
- **Instapaper Scraping & Utilities**:
  - `MacMiniM4/instapaper/instapaper.py` (Session scraper exporting docx/CSV)
  - `MacMiniM4/instapaper/instapaper1.py` (Hidden field authentication scraper)
  - `MacMiniM4/instapaper/quick.py` (HTML parse error checking script)
- **Project Subdirectories**:
  - `MacMiniM4/emailprocessing/`
  - `MacMiniM4/slack/`
  - `MacMiniM4/story generator/`
  - `MacMiniM4/video dedupe/`
  - `MacMiniM4/old_notused/`
