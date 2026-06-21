# Changelog

All notable changes to this template are documented here.

## [Unreleased]

### Fixed — request-copilot-review.yml: added --repo flag to gh pr edit; runner has no git checkout so repo context must be explicit. Added || true to prevent failure when Copilot is already assigned


### Fixed — request-copilot-review.yml: reverted Copi review request from curl REST call back to gh CLI; curl approach broke first-invocation auto-review


### Changed — BOOTSTRAP.md: PR directions restructured from A/B/C (Feature/Fix/Docs) to A/B (code/docs-tooling); hard stop rule, Copi gate, and day-to-day workflow updated to match current standard
### Changed — BOOTSTRAP.md: docs/tooling PR flow updated to skip Copi; matches CROG_ONBOARDING.md
### Changed — Copi gate rule: Copi review scoped to code PRs (`src/`) only; docs/tooling PRs skip Copi and go straight to Clead
### Changed — request-copilot-review.yml: workflow now only triggers on PRs touching `src/`; docs/tooling PRs no longer invoke Copi automatically
### Changed — pr_dump.sh: output wrapped in fenced code block so inline comment bodies render correctly in GitHub PR comments
### Added — Copi re-review known limitation documented in CROG_ONBOARDING.md

### Added
- Placeholder test files in each language pack — keeps CI green from day one

### Planned
- C# language pack (`languages/csharp/`) — dotnet test, dotnet format
- Elixir language pack (`languages/elixir/`) — mix test, ExUnit, Credo

## [1.0.0] — 2026-06-17

### Added
- Initial release of `ai-project-template`
- Bootstrap wizard (`BOOTSTRAP.md`) — five-phase guided setup via Claude chat
- AI team workflow: Clead (Claude chat), Crog (Claude Code CLI), Copi (GitHub Copilot Business, optional)
- Python language pack (`languages/python/`) — pytest, Ruff, pre-commit, GitHub Actions CI
- Node/TypeScript language pack (`languages/node/`) — Jest, Biome, GitHub Actions CI
- Language-agnostic core docs: TEAM_STRUCTURE, DEV_INFRASTRUCTURE, CROG_ONBOARDING, PRODUCT_BACKLOG templates
- `tools/dump.sh` — repo dump for Clead session catch-up
- `tools/pr_dump.sh` — PR review bundle for Clead (with `--no-src` flag for docs-only PRs)
- GitHub template repo enabled (`Use this template` button)
- Copi marked as optional — workflow functions with Clead as sole reviewer
