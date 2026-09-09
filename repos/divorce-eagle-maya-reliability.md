# LLM Overview — divorce
*Updated: 2026-06-05 21:51 UTC | Tier: standard | Auto-updated: daily cron*

## What This Is
**Read these in order at the start of every session:**

## Shareable Brief
This file is meant to be shared with another human or LLM when you want useful feedback without handing over the whole repository.

### What To Critique
- Whether the repo still matches its real job and current direction.
- What is overloaded, duplicated, or harder to understand than it should be.
- Which parts look safest to simplify, merge, extract, or delete.
- Which behavior is underspecified, brittle, or missing tests/docs.
- Whether the current architecture is still the right shape for the next phase.

### What To Ask
- What is this repo actually for right now?
- What seems overbuilt relative to its value?
- What is the first thing you would simplify if you inherited this tomorrow?
- What would you change if the repo had to support a new direction?
- What is the highest-risk assumption hiding in the current design?

### How To Use This File
- Give it to an LLM for architecture critique, simplification advice, or migration planning.
- Give it to a human for a compressed but still meaningful mental model.
- Treat it as a high-signal overview, not a substitute for the code or repo-local docs.

### Authority
- If this overview conflicts with code, `AGENTS.md`, `CLAUDE.md`, or local docs, the code and repo-local docs win.
- For exact inventories of jobs, routes, and connectors, use `docs/CAPABILITIES.md` when it exists.


## Current State
*Status: 🟢 active from local git history*

**Active work:**
- b3223cf1 feat(sync): Eagle Clio sync hardening + daily state commit
- 9b5a4664 feat(sync): eagle_daily_commit.sh — commit daily state to origin/main for stateless VMs
- 5d4e9456 fix(sync): commit .eagle_cache.json for stateless VM access; remove from gitignore

**Known issues:**
- No known issue found in recent commit subjects or local TODO/BLOCKERS docs.

**Recent changes (7 days):**
- `0f9cc08c log(session): CC session 20260605_125948 — 72 turns`
- `0df99547 log(session): CC session 20260605_121951 — 45 turns`
- `b5a8bd0a log(session): CC session 20260604_163617 — 174 turns`
- `5a975562 log(session): CC session 20260603_170839 — 343 turns`
- `c724b06d docs: finalize clio access and set four`
- `2a00a46b Merge remote-tracking branch 'origin/main' into codex/eagle-clio-sync-hardening`
- `b3223cf1 feat(sync): Eagle Clio sync hardening + daily state commit`
- `9b5a4664 feat(sync): eagle_daily_commit.sh — commit daily state to origin/main for stateless VMs`
- `5d4e9456 fix(sync): commit .eagle_cache.json for stateless VM access; remove from gitignore`
- `d4c93dab chore(daily): eagle state refresh 2026-06-03`
- `ff2403d6 sync: gmail 2026-06-03 13:20 [eagle_bg_sync]`
- `44a9bb2f log(session): CC session 20260603_131057 — 125 turns`
- `ca58a19a sync: gmail 2026-06-03 10:15 [eagle_bg_sync]`
- `bac8ddaa sync: gmail 2026-06-03 09:30 [eagle_bg_sync]`
- `a67e065a sync: gmail 2026-06-03 08:50 [eagle_bg_sync]`
- `49e7b2d4 sync: gmail 2026-06-02 22:00 [eagle_bg_sync]`
- `110bd8b6 sync: gmail 2026-06-02 20:50 [eagle_bg_sync]`
- `d6a0bcb5 sync: gmail 2026-06-02 20:05 [eagle_bg_sync]`
- `f78d387f sync: gmail 2026-06-02 14:10 [eagle_bg_sync]`
- `4f74bf9a docs(sync): explain Eagle trusted ingestion path`

## Architecture
- Stack marker: Makefile-driven operations
- Top-level entry: `1shot/`
- Top-level entry: `AGENTS.md`
- Top-level entry: `ARCHIVE/`
- Top-level entry: `CASE_BRIEF.md`
- Top-level entry: `CLAUDE.md`
- Top-level entry: `CONTEXT/`
- Top-level entry: `data/`
- Top-level entry: `divorce_discovery.db`

## Key Commands
- `git status --short`
- `git log --oneline -5`

## Dependencies
- **Runs on:** Not declared in local repo evidence.
- **Calls out to:** See repo docs and config files.
- **Called by:** Not declared in local repo evidence.
- **Env vars required:** No `.env.example` keys found.

## Critical Rules
- Preserve repo-local instructions in `AGENTS.md`, `CLAUDE.md`, or README when present.
- Do not infer behavior from the repository name alone; verify against local docs and source.

## Gotchas
- Generated from local evidence only: git history, top-level structure, README/CLAUDE/AGENTS/docs, and env examples.
