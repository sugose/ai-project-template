# Setup Checklist — ai-python-project-template

Complete these steps in order when starting a new project from this template.

## 1. Rename and configure

- [ ] Replace all `[PROJECT NAME]` placeholders across all docs
- [ ] Replace `[ORG]/[REPO]` with your actual GitHub org and repo name
- [ ] Replace `[PO NAME]` with the Product Owner's name
- [ ] Replace `[PRIMARY MACHINE]` and `[SECONDARY MACHINE]` with your machine descriptions
- [ ] Fill in the project-specific vocabulary table in `docs/CROG_ONBOARDING.md`
- [ ] Update `docs/PRODUCT_BACKLOG.md` with your first PBIs
- [ ] Update `tools/dump.sh` — replace `[PROJECT NAME]` and `[PO NAME]` in the session start instructions
- [ ] Create `docs/TECHNICAL_PRODUCT_SPECIFICATION.md` — this is Crog and Copi's authoritative reference

## 2. GitHub setup

- [ ] Create the GitHub repo (private or public)
- [ ] Push initial commit to master
- [ ] Create a GitHub Team organisation if branch protection enforcement on private repos is needed (free plan does not enforce on private repos)
- [ ] Create a branch protection ruleset `master protection` with: restrict deletions, require PR (0 approvals), require `build` CI status check, require branch up to date, block force pushes

## 3. Local environment

- [ ] Python 3.14.5 installed
- [ ] `python -m venv .venv`
- [ ] `.venv\Scripts\activate` (Windows) or `source .venv/bin/activate` (macOS)
- [ ] `pip install -r requirements.txt -r requirements-dev.txt`
- [ ] `pre-commit install`
- [ ] `pytest` — should pass with 0 tests (no tests yet)
- [ ] `ruff check src/` — should be clean

## 4. AI team setup

- [ ] Install Claude Code CLI: `npm install -g @anthropic-ai/claude-code`
- [ ] Authenticate Claude Code against your Claude Pro account
- [ ] Install GitHub Copilot Business extension in VS Code
- [ ] Verify Copilot is active (inline suggestions enabled)
- [ ] Open a Clead (Claude chat) session and paste the output of `bash tools/dump.sh`

## 5. VS Code layout (recommended)

- Clead in Simple Browser panel or separate window
- Crog in Claude Code panel
- Copi in Copilot Chat panel

## 6. Write your first spec

Before any code:
- [ ] Create `docs/TECHNICAL_PRODUCT_SPECIFICATION.md` — this is Crog and Copi's authoritative reference
- [ ] Add your first PBI to `docs/PRODUCT_BACKLOG.md`
- [ ] Hand the first task to Crog

## Notes

- Crog authors PRs under the account running `gh` CLI. GitHub forbids self-approval — set required approvals to 0 and use CI as the hard gate, with Clead review via `pr_dump.sh` as the quality gate.
- Node.js is required for Claude Code CLI.
- The `tools/dumps/` and `reviews/` folders are gitignored — they are working space, not versioned assets.
- `pyproject.toml` targets `py314` — adjust if using a different Python version.
