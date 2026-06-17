# ai-python-project-template

A template for AI-assisted Python projects using the Clead + Crog + Copi workflow.

## What This Is

This template packages a proven AI-assisted development workflow built around:

- **Clead** (Claude, chat interface) — Tech Owner: architecture, specs, PR review
- **Crog** (Claude Code, CLI) — Senior Developer: TDD-first implementation, autonomous git/PR workflow
- **Copi** (GitHub Copilot Business) — Code Reviewer: PR review via native GitHub integration
- **[PO NAME]** — Product Owner: direction, approvals, merge authority

The workflow is documented in full in `docs/TEAM_STRUCTURE.md`.

## What Is Included

| File/Folder | Purpose |
|---|---|
| `.github/workflows/ci.yml` | GitHub Actions CI: lint, format, test, coverage ≥80% |
| `.github/copilot-instructions.md` | Copi onboarding |
| `CLAUDE.md` | Crog auto-load entry point |
| `docs/CROG_ONBOARDING.md` | Crog's full mandate, workflow rules, code standards |
| `docs/TEAM_STRUCTURE.md` | Team roles, PR workflow, Clead Review Standard |
| `docs/DEV_INFRASTRUCTURE.md` | CI/CD, tooling, runbook |
| `docs/PRODUCT_BACKLOG.md` | Backlog template |
| `tools/dump.sh` | Repo dump for Clead session catch-up |
| `tools/pr_dump.sh` | PR review bundle for Clead |
| `pyproject.toml` | Ruff + pytest config |
| `requirements.txt` | Pinned runtime dependencies |
| `requirements-dev.txt` | Pinned dev dependencies |
| `.pre-commit-config.yaml` | Ruff pre-commit hooks |
| `.vscode/` | Shared editor settings |

## Getting Started

See `SETUP.md` for the complete first-time setup checklist.
