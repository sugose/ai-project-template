# GitHub Copilot Instructions — [PROJECT NAME]

**Before writing any code**, read `docs/TECHNICAL_PRODUCT_SPECIFICATION.md` — it is the authoritative source for component contracts and architecture.

## Project Structure

```
src/
  [module]/          # Main source modules
  tests/             # pytest — all tests fully mocked
docs/                # Specs, onboarding, strategy
tools/               # dump.sh, pr_dump.sh
```

## Essential Commands

```bash
pytest --cov=src --cov-fail-under=80
ruff check src/
ruff format --check src/
```

## Non-Negotiable Rules

- **Never commit to `master`** — work on `feature/<description>` branches, open PRs
- **TDD strictly** — write failing tests first, then implementation
- **No hardcoded values** — everything configurable
- **No direct `--no-verify` bypasses** — pre-commit hooks are enforced

## Code Standards

- **Python 3.14.5** — use modern stdlib (`collections.abc`, `ipaddress`, `pathlib.Path`)
- **Type hints** on all public functions; **docstrings** on all public functions/classes
- **Ruff** for linting (`line-length = 100`) and formatting
- **Exception chaining** — always `raise XxxError(...) from exc`

## Testing Rules

- No live network, filesystem, or external dependencies in tests — mock everything
- Test naming: `test_<unit>_<scenario>`
- Tests live in `src/tests/`
