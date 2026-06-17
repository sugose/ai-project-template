## Python Code Standards

- **Python 3.14.5** — use modern stdlib (`collections.abc`, `ipaddress`, `pathlib.Path`)
- **Type hints** on all public functions
- **Docstrings** on all public functions and classes — explain intent, not mechanics
- **Ruff** for linting (`line-length = 100`) and formatting — `ruff check src/` and `ruff format --check src/` must be clean
- **No hardcoded values** — everything configurable via config file or CLI
- **Exception chaining** — always `raise XxxError(...) from exc`
- **Virtual environment** — always use `.venv`; never install into system Python
- **Cross-platform paths** — `pathlib.Path` for all file paths, never string concatenation

### Running the project

```bash
# Activate environment
source .venv/bin/activate        # macOS
.venv\Scripts\activate           # Windows

# Run tests
pytest --cov=src --cov-fail-under=80

# Lint and format
ruff check src/
ruff format --check src/

# Run as module (always from project root)
python -m src.<module>.main
```

### Known platform gotcha

For multicast sending on Windows, `IP_MULTICAST_IF` must be set to the machine's actual IP — not `0.0.0.0`.
