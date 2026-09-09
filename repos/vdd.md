# LLM-OVERVIEW — vdd
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
`vdd` (Video Deduplication) is a local-only, privacy-first video content analysis, quality scoring, and deduplication system. It combines perceptual frame hashing (pHash, dHash, wHash), audio fingerprinting, OpenCV metadata extraction, and multi-stage filename/size matching to identify exact video duplicates, near duplicates, partial clips, and re-cuts across large media libraries. It integrates with Organized Operational Setup (OOS) context engineering, a Claude MCP server, and CLI tools for automated report and cleanup script generation.

## Machine & Host Ownership
- `homelab.yaml` registers project `vdd` (kind: `service`, lifecycle: `development`, systemd/runtime unit `vdd`) with target host specified as `unknown` managed by `unknown`.
- Operational execution targets local external storage paths such as `/Volumes/Storage` and `/Volumes/7TB/LRON` (e.g., `make full-analysis` targets `/Volumes/7TB/LRON`).
- No multi-host cluster configuration is declared in the repository evidence.

## What is actually built
- **`vdd/` Core Python Package**:
  - `vdd.core.db`: SQLite database layer (`VDDDatabase`) storing video metadata, perceptual hashes, audio fingerprints, and comparison results.
  - `vdd.helpers.hash`: `FrameHasher` executing pHash, dHash, and wHash algorithms via OpenCV (`cv2`) for visual similarity analysis.
  - `vdd.helpers.audio`: `AudioFingerprinter` for extracting and matching audio stream fingerprints.
  - `vdd.helpers.ff`: Helper utilities interfacing with `ffmpeg` and `ffprobe` for video decoding and audio stream extraction.
  - `vdd.cli.scan`: Core scanning logic powering the `vd` CLI driver (`vd scan`).
- **Deduplication & Analysis Runners**:
  - `vdd_analyzer.py` / `video_dedupe_analyzer.py`: Main batch analyzer. Extracts media attributes (resolution, bitrate, FPS, duration, codecs, frame count, audio presence), generates perceptual hashes, and logs `VideoAnalysisResult` entries to SQLite (`vdd_analysis.db`).
  - `vdd_dedup.py`: Relationship classification engine. Categorizes video pairs (`EXACT_DUPLICATE`, `NEAR_DUPLICATE`, `PARTIAL_CLIP`, `RECUT`, `DIFFERENT`) and recommends actions (`KEEP`, `DELETE`, `REVIEW`) based on quality scores (resolution, bitrate, codec, file size).
  - `two_phase_duplicate_finder.py`: Two-phase deduplicator writing to `media_duplicates.db` (`files` table). Phase 1 compares file hashes, normalized titles, and sizes; Phase 2 selectively runs video perceptual hashing on candidate pairs.
  - `fast_media_duplicate_finder.py`: Standalone duplicate finder combining exact file hashing, fuzzy string matching (`SequenceMatcher`), size matching, and regex tag normalization (stripping year, quality, and resolution markers).
- **OOS Context & MCP Server**:
  - `video_oos_mcp.py` (`make video-mcp`): Claude MCP server exposing video deduplication context engineering and analysis commands to LLM agents.
- **Cleanup Wrappers & Analysis Reports**:
  - Shell automation: `cleanup_video_duplicates.sh`, `run_cleanup_and_monitor.sh`, `run_full_analysis.sh`, and `cleanup_summary.txt`.
  - Report summaries stored in `results/` (`movies_duplicate_report.txt`, `tv_duplicate_report.txt`, `tv2_duplicate_report.txt`).
- **Dependencies & Configuration**:
  - Python dependencies (`requirements.txt`): OpenCV (`cv2`), SQLite (`sqlite3`), `tqdm`, `ffmpeg`/`ffprobe` external binaries.
  - Env vars: `ANTHROPIC_API_KEY`, `BACKUP_BEFORE_DELETE`, `CONTEXT7_API_KEY`, `DRY_RUN_DEFAULT`, `LOG_FILE`, `LOG_LEVEL`, `MAX_WORKERS`, `MCP_DEBUG_MODE`, `MCP_SERVER_PORT`, `PARALLEL_PROCESSING`, `VIDEO_BATCH_SIZE`, `VIDEO_INPUT_DIR`, `VIDEO_OUTPUT_DIR`, `VIDEO_QUALITY_THRESHOLD`.

## Canonical entry points
- `vd`: Primary CLI driver (`vd scan /Volumes/Storage --db output/vdd_analysis.db`).
- `python vdd_analyzer.py <DIR>`: Feature extraction and video analysis runner (`make analyze DIR=...`).
- `python vdd_dedup.py`: Deduplication relationship classification engine (`make dedup`).
- `python two_phase_duplicate_finder.py`: Two-phase file hash and perceptual hash duplicate finder.
- `python fast_media_duplicate_finder.py`: Fast multi-method string and file hash deduplicator.
- `Makefile`: Task runner (`make setup`, `make test`, `make analyze`, `make dedup`, `make full-analysis`, `make clean`).
- `python test_vdd.py` (`make test`): Verification test suite for database, audio fingerprinting, and frame hashing modules.
- `python video_oos_mcp.py` (`make video-mcp`): Claude MCP server entry point.
