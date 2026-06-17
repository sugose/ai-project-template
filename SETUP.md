# Manual Setup Checklist

Use this if you prefer to set up manually rather than using the bootstrap wizard in `BOOTSTRAP.md`.

## 0. Choose your language

This template supports:
- **Python** — uses pytest, Ruff, pip, pre-commit
- **Node/TypeScript** — uses Jest, Biome, npm

Copy the relevant files from `languages/python/` or `languages/node/` into your project root during setup.

## 1. Replace placeholders

Search and replace across all files:
- `[PROJECT NAME]` — your project name
- `[ORG]/[REPO]` — your GitHub org and repo name
- `[PO NAME]` — Product Owner name
- `[PROJECT DOMAIN]` — your domain/technology area
- `[PRIMARY MACHINE]` / `[SECONDARY MACHINE]` — your dev machines

## 2. GitHub repo

```bash
gh repo create [ORG]/[REPO] --private --clone
cd [REPO]
git add .
git commit -m "Initial commit"
git push origin main
```

## 3. Branch protection

Create a ruleset named `main protection` with:
- Restrict deletions
- Require PR before merging (0 required approvals)
- Require status check: `build`
- Require branch up to date before merge
- Block force pushes

Note: enforcement on private repos requires a GitHub Team organisation. Free plan does not enforce.

## 4. Local environment

**Python:**
```bash
python -m venv .venv
source .venv/bin/activate        # macOS
.venv\Scripts\activate           # Windows
pip install -r requirements.txt -r requirements-dev.txt
pre-commit install
pytest
ruff check src/
```

**Node/TypeScript:**
```bash
npm ci
npx biome ci .
npx jest
```

## 5. AI team

- Install Claude Code: `npm install -g @anthropic-ai/claude-code`
- Authenticate: `claude` (browser auth against Claude Pro account)
- Install GitHub Copilot extension in VS Code
- Verify Copilot licence is active

## 6. VS Code layout

- Clead in Simple Browser panel
- Crog in Claude Code panel
- Copi in Copilot Chat panel

## 7. First session

```bash
bash tools/dump.sh
```

Paste the output into a new Claude chat. Clead will orient itself and ask what today's work is.
