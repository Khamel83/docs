# LLM-OVERVIEW — kid-friendly-ai
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
Kid-Friendly AI Buddy (`buddy.khamel.com`) is a safe, interactive web application and educational companion for children aged 6–12. Built with Next.js 14, React 18, and TypeScript, it integrates voice-first conversational AI, multi-language speech capabilities, content moderation tailored for young users, dual-mode AI inference (local Mac Mini vs. cloud models), and interactive learning mini-games.

## Machine & Host Ownership
- **Homelab Contract**: Registered under `homelab.yaml` as project `kid-friendly-ai` (kind: service, lifecycle: development, owner: homelab, standby monitoring, systemd unit: `kid-friendly-ai`).
- **Mac Mini (Local Inference Host)**: Runs `macmini/piper-server.py` (FastAPI / Uvicorn server providing local Piper TTS with CORS support for `buddy.khamel.com`) alongside a local Ollama LLM engine.
- **Cloud / Vercel Host**: Configured for edge deployment (`verify-deployment.js` target `https://kid-friendly-ai.vercel.app`, environment variable `NEXT_PUBLIC_VERCEL_URL`).
- **Docker / Systemd Hosts**: Containerized via `docker-compose.yml` (`kid-friendly-ai-app` container exposing port 3000) and managed on host systems via `buddy.service` systemd configuration.
- **AWS Infrastructure**: Provisioning templates contained in `aws/` (CloudFormation, Terraform, and `aws/deploy-aws.sh`).

## What is actually built
- **Next.js 14 Application & API Core**: Frontend UI and serverless routes built on Next.js 14.0.4, React 18.2.0, TypeScript, and Redux Toolkit. Features PWA support (`/manifest.json`, `/sw.js`) and API health checking (`/api/health`).
- **Dual-Mode AI Engine**:
  - *Cloud Pipeline*: Integrates OpenRouter API (`google/gemini-2.5-flash-lite` default model, configurable temperature/token caps) and OpenAI API (`gpt-4.1-nano`).
  - *Local Pipeline*: Communicates with Mac Mini Ollama local LLM and Piper TTS.
  - *TTS Integration*: ElevenLabs streaming TTS (`elevenlabs-tts.ts`) with child-friendly voices, usage tracking, and sentence streaming.
- **Real-Time Audio Worklet**: `public/enhanced-audio-processor.js` implements an `AudioWorkletProcessor` delivering real-time noise reduction, voice threshold detection, gain management, and Voice Activity Detection (VAD). Includes click-to-talk toggle, voice card controls, and auto-silence detection.
- **Educational Mini-Games**:
  - *Space Explorer* and *Emoji Detective* games with interactive question streams.
  - *Pattern Puzzle* (`NEXT_PUBLIC_ENABLE_PATTERN_PUZZLE`) and *Animal Quiz* (`NEXT_PUBLIC_ENABLE_ANIMAL_QUIZ`).
  - Reward system featuring interactive sticker collection.
- **Safety & Parental Controls**: Content filtering middleware, age 7 prompt constraints, COPPA/GDPR compliance hooks, and configurable feature flags (`NEXT_PUBLIC_ENABLE_VOICE_INPUT`, `NEXT_PUBLIC_ENABLE_PARENTAL_CONTROLS`, `NEXT_PUBLIC_ENABLE_SOUND_EFFECTS`).
- **Health Checks & Telemetry**: Standalone Docker health script (`healthcheck.js` / `docker/healthcheck.js`) monitoring memory limit (500MB), CPU load (80%), HTTP readiness, and optional Redis connectivity. Observability events stream to `homelab-api:/observations`.

## Canonical entry points
- **Next.js Web Server**: `package.json` scripts (`npm run dev`, `npm run build`, `npm run start`).
- **Framework Configuration**: `next.config.js` handling strict mode, SWC minification, and optional bundle analysis (`ANALYZE=true`).
- **Local TTS Server**: `macmini/piper-server.py` (FastAPI app running local Piper TTS models from `voices/`).
- **Container Specs**: `docker-compose.yml`, `docker/docker-compose.prod.yml`, and `Dockerfile`.
- **Systemd Daemon**: `buddy.service` host service specification.
- **Deployment & Verification**: `deploy.sh`, `deploy-docker.sh`, `aws/deploy-aws.sh`, and `verify-deployment.js`.
- **Audio Worklet Runtime**: `public/enhanced-audio-processor.js`.
- **Test Suites**: `npm run test` (Jest unit/component tests in `__tests__/`), `npm run e2e` (Playwright E2E tests), and `npm run audit` (Lighthouse performance checks).
