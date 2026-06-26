# Clead–Crog Routines Pipeline

## Overview

Clead and Crog each run as Claude Code Routines at `claude.ai/code/routines`.

- **Clead** is the Tech Owner. No repositories are attached to Clead's Routine. Clead drafts task specifications and architectural guidance.
- **Crog** is the Task Executor. All project repositories are attached to Crog's Routine. Crog implements, commits, opens PRs, and reports back.

---

## How It Works

1. Adam fires Clead via `curl`, including `ADAM-AUTH` in the `text` field.
2. Clead drafts a task specification and produces a ready-to-fire `curl` command targeting Crog's Routine, with `CROG_TOKEN` as a placeholder for the bearer token.
3. Adam replaces `CROG_TOKEN` with the real token from his password manager and fires Crog.
4. Crog executes the task: implements the change, opens a PR against `main`, posts the `pr_dump` output as a PR comment, and reports the PR URL back.
5. Adam relays the PR URL to Clead for review. The normal PR review flow continues from there.

---

## Authentication

- All requests to Clead must include `ADAM-AUTH` in the `text` field. Requests without it are rejected.
- Crog tokens are stored in a password manager. They are never stored in repository files or standing prompts.

---

## Token Management

- Each Routine has its own bearer token.
- Tokens are shown **once** at generation time — store immediately in your password manager.
- Regenerating a token invalidates the previous one.
- Never embed tokens in standing prompts or commit them to git.

---

## Current Limitations

Routines do not yet support secrets injection — tokens cannot be provided to a Routine as environment variables. This means Clead cannot fire Crog autonomously; Adam must relay the `curl` command and substitute the token manually.

Full autonomy (Clead fires Crog directly) is the target once Anthropic adds secrets support to Routines.

---

## Curl Command Pattern

Requests to a Routine use a single-line `curl` with the `-s` flag for clean output. The `anthropic-beta` header `experimental-cc-routine-2026-04-01` is required.

```bash
curl -s -X POST https://api.anthropic.com/v1/claude_code/routines/<ROUTINE_ID>/fire -H "Authorization: Bearer <TOKEN>" -H "anthropic-version: 2023-06-01" -H "anthropic-beta: experimental-cc-routine-2026-04-01" -H "Content-Type: application/json" -d "{\"text\": \"<TASK_SPECIFICATION>\"}"
```

Replace `<ROUTINE_ID>`, `<TOKEN>`, and `<TASK_SPECIFICATION>` with real values. Never store the token anywhere other than your password manager.
