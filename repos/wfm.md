# LLM-OVERVIEW — wfm
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
The `wfm` repository houses the USC Workforce Management (WFM) Hub — an employee-centric workforce case management, reporting analytics, and multi-system HR claims validation platform designed for USC academic schools and business units. The repository contains Flask web applications, structured JSON/CSV data storage for cases and cross-system claims, a drop-in `tailscale-funnel-module` for zero-cost secure local web hosting over Tailscale Funnel HTTPS, and homelab governance specifications (`homelab.yaml`, `AGENTS.md`).

## Machine & Host Ownership
- **Deployment Runtime Host**: `homelab.yaml` declares service `wfm` under schema `homelab.project/v1` with lifecycle `development`, monitoring state `standby`, and runtime target `production.host: unknown` (`managed_by: unknown`, unit `wfm`). No single dedicated production server IP or host is bound in active code configurations.
- **Worker Dispatch Remote Hosts**: `AGENTS.md` defines SSH worker dispatch hosts `oci-ts` and `macmini-ts` for multi-model task execution (`ssh oci-ts ...`, `ssh macmini-ts ...`).
- **Search Infrastructure**: Web search operations route through the homelab Argus service at `http://100.112.130.100:8270/api/search`.
- **Status Probe**: Live status probe is disabled by default (`JANITOR_RUN_STATUS_PROBE=1` required to execute `scripts/status.py`).

## What is actually built
- **Core Application Suites**:
  - `WFM/app.py`: Minimal Flask deployment server running on `0.0.0.0:5000` (configurable via `PORT`), serving system readiness status and Railway deployment status (replacing retired Vercel serverless deployment).
  - `WFM/pythonanywhere_app.py`: Full-featured Flask application backed by HTML templates in `WFM/templates/`, exposing routes `/` (dashboard), `/reporting` (analytics), `/case/<case_id>` (case details), `/intake` (case submission), `/source_tables` (source table views), `/import_ms_forms` (Microsoft Forms importer), and `/download_table/<table_name>` (CSV downloader).
  - `usc_wfm_final/app.py`: Standalone single-file Flask application utilizing embedded Jinja2 HTML templates via `render_template_string`, running on port `9000` (or `PORT`), exposing `/` (home dashboard), `/analytics` (charts & statistics), `/cases` (case index), `/case/<case_id>` (details & claims check trigger), and `/claims` (5-system HR lookup).
  - `WFM/simple_app.py`: Simple Flask deployment test stub.
  - `WFM/api/index.py` & `WFM/api/hello.py`: Retired Vercel serverless function entry points.
- **Data Model & Cross-System Claims Architecture**:
  - `WFM/data/cases.json` & `usc_wfm_final/data/cases.json`: Primary JSON document database storing cases with attributes `case_id`, `title`, `school_unit` (covering 21 USC academic schools and 15 business units), `request_type`, `status`, `created_date`, `priority`, `description`, and assigned `employees` array (`name`, `employee_id`, `department`).
  - `WFM/data/source_tables/`: CSV datasets representing 5 integrated HR compliance domains: `ER_OPE_export.csv` (Employee Relations), `OGC_export.csv` (Office of General Counsel), `ADA_export.csv` (Accessibility & Accommodations), `Workers_Comp_export.csv` (Workers Compensation), and `TAB_TE_export.csv` (Time & Attendance), alongside `cases_export.csv` and `sample_employees.csv`.
  - `WFM/generate_comprehensive_test_data.py`: CLI generator script constructing 40-50 synthetic cases across USC organizational units.
  - `WFM/debug_claims_check.py` & `WFM/test_claims_check_real.py`: Execution scripts for testing step-by-step claims verification against source table DataFrames using employee IDs (e.g. `12345678`).
- **Tailscale Funnel Web Hosting Module (`tailscale-funnel-module/`)**:
  - Shell automation scripts (`scripts/`): `funnel-setup.sh` (environment setup), `funnel-start.sh` (launches local app and Funnel process), `funnel-stop.sh` (stops Funnel process), `funnel-status.sh` (queries tunnel status and public `.ts.net` URL), `funnel-watchdog.sh` (process restart supervisor), `multi-project.sh` (multi-port proxy management), and `setup-tailscale-mcp.sh` (MCP setup).
  - Environment templates & configurations (`templates/`): `.env.tailscale.example`, `tailscale-config.json`, and sample applications (Flask, Express, Next.js).
  - Model Context Protocol (MCP) integrations: `.mcp.json.http-local`, `.mcp.json.http-public`, and `.mcp.json.stdio`.
  - Deployment root launcher: `start_app.sh` (checks `$BASE_URL`, prints default credentials `admin`/`admin123`, and starts Flask on `0.0.0.0:5000`).
- **Governance & Orchestration**:
  - `homelab.yaml`: Project registration spec (`homelab.project/v1`, project `wfm`, owner `homelab`, lifecycle `development`, monitoring `standby`).
  - `AGENTS.md`: ONE_SHOT v14 Orchestration Operating Contract defining task categories, routing tables (`claude_code`, `codex`, `gemini_cli`, `glm_claude`/`zai`, `free`), dispatch protocols, and Argus search API (`http://100.112.130.100:8270/api/search`).

## Canonical entry points
- **Start Application (Tailscale Funnel / Local)**: `./start_app.sh`
- **Run Railway / Minimal Flask Server**: `python3 WFM/app.py`
- **Run Template-Based Flask App**: `python3 WFM/pythonanywhere_app.py`
- **Run Standalone USC WFM Final Server**: `python3 usc_wfm_final/app.py`
- **Tailscale Funnel Setup & Launch**: `./tailscale-funnel-module/scripts/funnel-setup.sh` && `./tailscale-funnel-module/scripts/funnel-start.sh`
- **Check Tailscale Funnel Status**: `./tailscale-funnel-module/scripts/funnel-status.sh`
- **Generate Synthetic Test Data**: `python3 WFM/generate_comprehensive_test_data.py`
- **Run Real Claims Verification Test**: `python3 WFM/test_claims_check_real.py`
- **Debug Claims Verification Engine**: `python3 WFM/debug_claims_check.py`
- **ONE_SHOT Routing Resolution**: `python3 -m core.router.resolve --class implement_small --category coding`
