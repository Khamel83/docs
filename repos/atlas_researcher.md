# LLM-OVERVIEW — atlas_researcher
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
Atlas Researcher is a Next.js 15 multi-agent AI research application that automates end-to-end web research using OpenRouter model endpoints. The system orchestrates four specialized AI agents (Planner, Searcher, Evaluator, Synthesizer) to decompose questions, search web sources, evaluate source credibility, and generate cited markdown research reports with phase-based session recovery and configurable research depth modes.

## Machine & Host Ownership
No multi-host deployment runtime information is specified in status probes or configuration (`homelab.yaml` sets production host and manager to `unknown`, while `vercel.json` provides optional Vercel deployment configuration).

## What is actually built
- **Multi-Agent Research Pipeline (`src/lib/agents/`)**:
  - `PlannerAgent` (`planner.ts`): Decomposes complex user research queries into targeted subtopics.
  - `SearcherAgent` (`searcher.ts`): Queries web search APIs to fetch subtopic source material.
  - `EvaluatorAgent` (`evaluator.ts`): Rates source credibility, filters noise, and extracts key facts/evidence (`EvaluationResult`).
  - `SynthesizerAgent` (`synthesizer.ts`): Integrates evaluated evidence into structured markdown reports (`ResearchReport`) with formatted citations and section metadata.
- **Model Orchestration & Router (`src/lib/`)**:
  - `openrouter.ts`: OpenRouter API client integration (`createOpenRouterClient`).
  - `models.ts`: Model selector (`modelRouter`) assigning available free/paid LLMs to pipeline phases based on task requirements.
- **Session State & Recovery (`src/lib/research-session.ts`)**:
  - `ResearchSession`: Schema holding question, status (`pending`, `planning`, `searching`, `evaluating`, `synthesizing`, `completed`, `failed`), progress percentage, token usage, subtopics, evaluated sources, and intermediate phase results (`planningResult`, `searchResults`, `evaluationResults`, `synthesisResult`).
  - Progressive auto-saving (every 30 seconds) allowing research sessions to resume from any phase following timeouts or interruptions.
- **Report & Storage Engine (`src/lib/storage.ts`, `src/lib/storage-vercel.ts`)**:
  - `ReportStorage`: Manages generation and indexing of timestamped `.md` reports and metadata (`public/reports/index.json` containing filename, timestamp, query, word count, and models used).
  - `VercelReportStorage`: Alternative storage driver for Vercel deployment compatibility.
- **API Server Routes (`src/app/api/`)**:
  - `/api/research` (`POST`): Research pipeline handler (`executeResearchWithProgression`) supporting `normal` (10 sources/subtopic) and `max` (30+ sources/subtopic) research depth modes with progress updates.
  - `/api/reports` (`GET`): Paginated index endpoint for generated reports (supports `limit` param up to 100).
  - `/api/reports/[filename]` (`GET`): Individual report retrieval endpoint with directory traversal validation (`^[\d\w\-_]+\.md$`).
- **Project Operations & Setup**:
  - `homelab.yaml`: Project declaration (`homelab.project/v1`, project `atlas-researcher`, standby monitoring state).
  - `bin/setup-1password-service.sh` & `bin/setup-archon.sh`: Credential management and development workspace setup scripts.
  - `start-production.sh`: Launch script for production deployments.

## Canonical entry points
- `npm run dev`: Starts local Next.js development server with Turbopack (`next dev --turbopack`).
- `npm run build`: Compiles production build with Turbopack (`next build --turbopack`).
- `npm run start`: Runs Next.js production server (`next start`).
- `npm run lint`: Runs ESLint checks (`eslint`).
- `src/app/api/research/route.ts`: Entry route for executing research jobs.
- `src/app/api/reports/route.ts`: API route for listing research reports index.
- `src/app/api/reports/[filename]/route.ts`: API route for serving specific report content.
- `start-production.sh`: Shell script launching production execution.
