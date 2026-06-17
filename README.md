# ai-project-template

A template for AI-assisted software projects using the Clead + Crog + Copi workflow.

## Supported Languages

| Language | Test framework | Linter/Formatter |
|---|---|---|
| Python 3.14.5 | pytest | Ruff |
| Node/TypeScript 22 | Jest + ts-jest | Biome |

## What This Is

This template packages a proven AI-assisted development workflow for software projects. It includes all tooling, documentation templates, and AI agent onboarding needed to start building immediately.

The workflow uses three AI agents with distinct roles:

| Agent | Tool | Role |
|---|---|---|
| Clead | Claude (chat) | Tech Owner — architecture, specs, PR review |
| Crog | Claude Code (CLI) | Senior Developer — TDD implementation, autonomous git/PR |
| Copi | GitHub Copilot Business | Code Reviewer — PR review via GitHub integration |

## Getting Started

**The fastest way to set up a new project from this template:**

1. Open a fresh Claude chat at [claude.ai](https://claude.ai)
2. Open `BOOTSTRAP.md` in this repo
3. Copy everything below the divider line and paste it into the chat
4. Follow Clead's questions — your project will be fully set up by the end

The bootstrap wizard will walk you through:
- Choosing your language (Python or Node/TypeScript)
- Collecting your project details
- Generating all project documentation with your specifics filled in
- Setting up the GitHub repo via Crog
- Configuring branch protection
- Writing your first technical spec
- Handing your first task to Crog

## What Is Included

| File/Folder | Purpose |
|---|---|
| `BOOTSTRAP.md` | Paste-and-go setup wizard prompt for Claude |
| `CLAUDE.md` | Crog auto-load entry point |
| `languages/python/` | Python-specific CI, tooling, and code standards |
| `languages/node/` | Node/TypeScript-specific CI, tooling, and code standards |
| `.github/copilot-instructions.md` | Copi onboarding template |
| `docs/CROG_ONBOARDING.md` | Crog's full mandate and workflow rules |
| `docs/TEAM_STRUCTURE.md` | Team roles, PR workflow, Clead Review Standard |
| `docs/DEV_INFRASTRUCTURE.md` | CI/CD, tooling, runbook |
| `docs/PRODUCT_BACKLOG.md` | Backlog template |
| `tools/dump.sh` | Repo dump for Clead session catch-up |
| `tools/pr_dump.sh` | PR review bundle for Clead |

## Prerequisites

- Node.js (for Claude Code CLI)
- Claude Pro subscription (covers both Clead and Crog)
- GitHub Copilot Business licence
- Git + GitHub CLI (`gh`)
- For Python projects: Python 3.14.5
- For Node projects: Node.js 22 LTS

See `SETUP.md` for the manual setup checklist if you prefer not to use the bootstrap wizard.
