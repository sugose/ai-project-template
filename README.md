# ai-project-template

A template for AI-assisted software projects using the Clead + Crog + Copi workflow.

## Use this template

Click the **Use this template** button at the top of this repo on GitHub to create a new repo pre-populated with all template files — no cloning or manual setup needed.

Then open `BOOTSTRAP.md` in your new repo, copy everything below the divider line, and paste it into a fresh Claude chat. Clead will walk you through the rest.

---

## The AI Team

This template is built around three AI agents working together with a human Product Owner. Each has a distinct role, a distinct tool, and a distinct place in the workflow. Together they cover the full development lifecycle — from architecture to implementation to review — without any of them needing to do everything.

### Clead — Tech Owner
**Tool:** Claude (chat interface at claude.ai)

Clead is the architect. It owns the technical direction of the project: designing the system, writing the specification, reviewing every pull request, and making the final call on all technical decisions. Clead never writes directly to the repo — it works through the chat interface, with the Product Owner acting as its hands in GitHub.

Clead reviews every PR via a tool called `pr_dump.sh`, which bundles the full diff and source context into a single file that gets pasted into the chat. Clead's review follows a five-point standard: threat model, TPS compliance, error handling pass, test quality check, and an explicit list of what was not checked.

At the start of every session, the Product Owner runs `tools/dump.sh` and pastes the output into a fresh Claude chat. Clead reads the full repo state and picks up exactly where the last session left off.

### Crog — Senior Developer
**Tool:** Claude Code (CLI, runs in the terminal)

Crog is the implementer. It works directly in the repo — creating branches, writing tests, writing code, committing, pushing, and opening pull requests autonomously via the `gh` CLI. The Product Owner sets the task; Crog executes it without needing to be walked through every command.

Crog follows TDD strictly: tests first (red), then implementation (green), then refactor. It never commits directly to `master`. Every piece of work lands via a pull request that Clead reviews.

Crog is not a passive code generator. It raises concerns, flags spec gaps, pushes back on unnecessary complexity, and surfaces alternatives. It speaks up when something is worth raising and implements cleanly when it is not.

### Copi — Code Reviewer (optional)
**Tool:** GitHub Copilot Business (VS Code + native GitHub PR integration)

Copi is an optional but recommended addition to the team. It reads full file context — not just diffs — which means it catches cross-section inconsistencies that Clead, reviewing from a diff only, can miss. Copi reviews pull requests directly in GitHub via the native Copilot code review integration.

If you have a GitHub Copilot Business licence, add Copi as a standard reviewer on all significant code PRs. If not, Clead carries the full review load — the workflow functions without Copi, but the three-layer review catches more than two.

---

## Why This Workflow Works

Each agent has a structural advantage the others lack:

- **Clead** has the full project context in its chat window and the spec in its head. It catches architectural drift and spec deviations.
- **Crog** has direct repo access and terminal execution. It moves fast, catches nothing from the diff that Clead doesn't, but finds real implementation bugs through test execution.
- **Copi** (optional) reads full files. It finds bugs that only appear when you see a function and its callers together. Without Copi, run `pr_dump.sh` with the `--no-src` flag disabled to give Clead the fullest possible context.

No single agent catches everything. The three-layer review catches most things. The Product Owner — a human with domain knowledge and final authority — catches the rest.

---

## Supported Languages

| Language | Test framework | Linter/Formatter |
|---|---|---|
| Python 3.14.5 | pytest | Ruff |
| Node/TypeScript 22 | Jest + ts-jest | Biome |

---

## What Is Included

| File/Folder | Purpose |
|---|---|
| `BOOTSTRAP.md` | Paste-and-go setup wizard prompt for Claude |
| `CLAUDE.md` | Crog auto-load entry point |
| `.github/copilot-instructions.md` | Copi onboarding |
| `docs/CROG_ONBOARDING.md` | Crog's full mandate, workflow rules, code standards |
| `docs/TEAM_STRUCTURE.md` | Team roles, PR workflow, Clead Review Standard |
| `docs/DEV_INFRASTRUCTURE.md` | CI/CD, tooling, runbook |
| `docs/PRODUCT_BACKLOG.md` | Backlog template |
| `languages/python/` | Python-specific CI, tooling config, code standards |
| `languages/node/` | Node/TypeScript-specific CI, tooling config, code standards |
| `tools/dump.sh` | Repo dump for Clead session catch-up |
| `tools/pr_dump.sh` | PR review bundle for Clead |

---

## Prerequisites

**Required:**
- A Claude Pro subscription (covers both Clead and Crog)
- Node.js (for Claude Code CLI: `npm install -g @anthropic-ai/claude-code`)
- Git + GitHub CLI (`gh auth login`)
- Python 3.14.5 or Node.js 22 depending on your chosen language

**Optional but recommended:**
- GitHub Copilot Business licence — adds Copi as a second code reviewer with full-file context

See `SETUP.md` for the manual setup checklist, or use `BOOTSTRAP.md` for the guided wizard.
