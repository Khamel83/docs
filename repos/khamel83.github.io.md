# LLM-OVERVIEW — khamel83.github.io
> Current compressed briefing. Updated 2026-09-08. Agent behavior is defined in `AGENTS.md`. This derived file is not an independent authority.

## What this repo is
`khamel83.github.io` is a GitHub Pages static site repository and Homelab service endpoint. It hosts personal static web content and tracks service governance under the `homelab.project/v1` contract. A retirement Monte Carlo web application was briefly committed (`a97e4fc`) but subsequently reverted (`3354851`), maintaining the repo's minimal static site baseline.

## Machine & Host Ownership
No multi-host or explicit machine deployment information is available; configuration specifies `production.host: unknown` and `managed_by: unknown`.

## What is actually built
- **Homelab Project Metadata (`homelab.yaml`)**:
  - **Contract**: `homelab.project/v1` spec with project ID `khamel83-github-io`, owned by `homelab` under `development` lifecycle.
  - **Monitoring & Telemetry**: Monitoring state set to `standby`; telemetry points to `homelab-api:/observations` event sink with `sanitized` evidence policy.
  - **Dependencies & Security**: Zero required or optional external dependencies; registered secret broker consumer `khamel83-github-io` with no active secret references.
  - **Documentation Pointer**: Declares index as `README.md` and operations path as `docs/OPERATIONS.md`.
- **Static Web Subsystem**:
  - `index.html`: Main HTML entry point for the GitHub Pages site.
  - `CNAME`: GitHub Pages custom domain routing file.
  - `info2`: Supplementary static text/content file.
  - `README.md`: Project title anchor file.
- **Retired History**:
  - Retirement Monte Carlo web application (added in `a97e4fc`, reverted in `3354851`).

## Canonical entry points
- Website root: `index.html`
- Domain routing: `CNAME`
- Service contract: `homelab.yaml`
- Repository documentation: `README.md`
- Auxiliary data asset: `info2`
