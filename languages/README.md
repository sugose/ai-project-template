# Language Packs

Each subdirectory here is a self-contained language pack. The bootstrap wizard reads these packs to generate the correct CI pipeline, tooling config, and code standards for a new project.

## Structure of a language pack

Every language pack must contain exactly these files:

| File | Purpose |
|---|---|
| `ci.yml` | GitHub Actions CI pipeline for this language |
| `code-standards.md` | Language-specific code standards — appended to `docs/CROG_ONBOARDING.md` by the wizard |
| `vscode-settings.json` | VS Code settings for this language |
| `vscode-extensions.json` | Recommended VS Code extensions |
| `gitignore` | Language-specific `.gitignore` entries |
| Language tooling files | e.g. `pyproject.toml` + `requirements.txt` for Python; `package.json` + `tsconfig.json` + `biome.json` for Node; `package.json` + `app.json` + `tsconfig.json` + `biome.json` for react-native |
| `test_placeholder.*` | Single always-passing test — keeps CI green on day one; deleted when first real test is written |

## How to add a new language pack

1. Create a new subdirectory: `languages/<lang>/`
2. Add all required files listed above
3. Update `BOOTSTRAP.md`:
   - Add the new language to Question 2's option list
   - Add a new branch in Phase 2 doc generation for the language-specific code standards
   - Add a new Crog setup prompt branch in Phase 3
4. Update `README.md`: add a row to the Supported Languages table
5. Update `CHANGELOG.md`: add an entry under a new version
6. Open a PR — the PR template will guide you through what to include

## Existing language packs

| Pack | Test framework | Linter/Formatter | CI |
|---|---|---|---|
| `python/` | pytest + pytest-cov | Ruff | GitHub Actions (ubuntu-latest, Python 3.14.5) |
| `node/` | Jest + ts-jest | Biome | GitHub Actions (ubuntu-latest, Node 22) |
| `react-native/` | jest-expo + @testing-library/react-native | Biome | GitHub Actions (ubuntu-latest, Node 22, TypeScript check + unit tests only — no device build) |

## Notes

- Language pack files are not committed directly to a new project repo. The bootstrap wizard reads their contents and generates the files in the target project.
- The `ci.yml` in each pack becomes `.github/workflows/ci.yml` in the new project.
- The `gitignore` in each pack becomes `.gitignore` in the new project (merged with the root `.gitignore` from this template).
- `vscode-settings.json` and `vscode-extensions.json` become `.vscode/settings.json` and `.vscode/extensions.json`.
