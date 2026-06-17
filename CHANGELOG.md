# Changelog

All notable changes to this template are documented here.

## [Unreleased]

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
