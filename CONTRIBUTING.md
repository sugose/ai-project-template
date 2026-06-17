# Contributing to ai-project-template

This template is itself maintained using the Clead + Crog + Copi workflow it describes. Contributions follow the same process.

## How to suggest an improvement

**For bugs or gaps in the template:**
Open a GitHub Issue using the "Bug in template" issue template. Describe what is wrong, what you expected, and which file is affected.

**For new language pack requests:**
Open a GitHub Issue using the "New language pack request" issue template. Describe the language, preferred test framework, and preferred linter/formatter.

**For pull requests:**
1. Fork the repo or create a branch
2. Make your changes
3. Open a PR — the PR template will guide you through what to include
4. A maintainer will review

## How to add a language pack

See `languages/README.md` for the full guide.

## Principles

- **No half-steps** — documentation only commits when the work it describes is fully executed
- **Language packs are self-contained** — all language-specific files live in `languages/<lang>/`; nothing language-specific belongs in the repo root or core docs
- **The wizard is the product** — any change to a language pack must be reflected in `BOOTSTRAP.md`
- **Backwards compatibility** — changes to core docs templates should not break projects already built from this template without a clear migration note in `CHANGELOG.md`
