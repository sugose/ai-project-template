# [PROJECT NAME] Team Structure & Workflow

**Date:** [DATE]
**Last Updated:** [DATE]
**Version:** 1.0
**Status:** Current

---

## 1. Team Composition

| Role | Member | Environment |
|------|--------|-------------|
| Product Owner | [PO NAME] | [PRIMARY MACHINE] (primary), [SECONDARY MACHINE] (optional) |
| Tech Owner / Architect / Code Reviewer | Clead (Claude, chat) | Browser (claude.ai) on [PRIMARY MACHINE] |
| Senior Developer | Crog (Claude Code, CLI) | Terminal on [PRIMARY MACHINE] |
| Code Reviewer (optional) | Copi (GitHub Copilot Business) | VS Code + GitHub PR integration — add if Copilot Business licence is available |
| Code Reviewer (optional) | [OPTIONAL: human reviewer] | Direct GitHub access |
| CEO | [OPTIONAL: mascot] | [Location] |

---

## 2. Roles & Responsibilities

### [PO NAME] — Product Owner
- Sets product direction and priorities
- **Holds ultimate decision-making authority** — has final say on all matters, technical or otherwise
- Stays hands-off on technical decisions in practice, deferring to Clead on architecture
- Arbitrates if Clead and Crog disagree on a technical matter

**Acting as Clead's hands in GitHub (Clead has no direct access):**

Clead cannot access GitHub, the file system, or the terminal directly. [PO NAME] acts as the bridge between Clead and the rest of the team. The following tasks fall to [PO NAME] on Clead's behalf:

| Task | Git / GitHub command |
|---|---|
| Create a feature branch for Clead's work | `git checkout -b feature/<description>` |
| Stage files for commit | `git add <file>` or `git add .` |
| Commit Clead's code | `git commit -m "<message>"` |
| Push branch to GitHub | `git push origin feature/<description>` |
| Open a PR on Clead's behalf | GitHub UI — New Pull Request |
| Get a diff for Clead to review | Run `tools/pr_dump.sh` and paste output into Clead's chat |
| Post Clead's review feedback on a PR | GitHub UI — Add comment on PR |
| Post Clead's approval on a PR | GitHub UI — Approve PR (posted as comment since Clead has no GitHub account) |
| Relay reviewer inline PR comments to Clead | Copy comment text from GitHub PR, paste into Clead's chat |
| Merge an approved PR | GitHub UI — Merge Pull Request |
| Pull latest main after merge | `git checkout main && git pull` |

Crog (Claude Code) has direct terminal and `gh` CLI access — Crog manages its own branches, commits, pushes, and PR creation without [PO NAME] as relay.

### Clead — Tech Owner / Architect / Code Reviewer
- Owns technical architecture and system design decisions
- **Holds final technical authority** on architectural choices, subject only to [PO NAME]'s override
- Defines patterns, structures, and conventions for the codebase
- Produces architectural designs, documentation, and technical specifications
- Reviews code produced by Crog before it is merged — primary reviewer on all Crog PRs
- Identifies logic issues, edge cases, performance concerns, and maintainability problems
- Provides the reasoning and rationale behind all technical choices
- Receives and considers implementation feedback from Crog on architectural proposals
- Works from context provided in chat — has no direct file system or GitHub access

### Crog — Senior Developer (Claude Code)
- Implements features and writes code directly via the terminal (Claude Code CLI)
- Has direct terminal, git, and `gh` CLI access — manages branches, commits, pushes, and PRs independently
- Executes the architectural direction set by Clead
- Produces code subject to Clead's review before merging
- Follows TDD strictly: tests first (red), implementation (green), refactor
- Raises concerns, suggests alternatives, and pushes back when warranted — see `docs/CROG_ONBOARDING.md`
- Never commits directly to `main`; all work lands via Pull Request

---

## 3. Machines

| Machine | OS | Primary Use |
|---------|----|-------------|
| [PRIMARY MACHINE] | [OS] | Primary development — Clead (browser), Crog (terminal), Git |
| [SECONDARY MACHINE] | [OS] | [PURPOSE — e.g. field kit, testing, secondary dev] |

---

## 4. Day-to-Day Workflow

1. [PO NAME] picks the next PBI from `docs/PRODUCT_BACKLOG.md`.
2. [PO NAME] pastes the Crog task prompt into Claude Code.
3. Crog implements, opens a PR, waits for Copi to complete its review (code PRs only — docs/tooling PRs skip Copi), runs `pr_dump.sh`, and reports back.
4. Clead reviews in Claude chat.
5. [PO NAME] merges on Clead's approval.
6. [PO NAME] updates `CHANGELOG.md` and moves to the next PBI.

Clead and Crog are separate instances with separate contexts — Clead sets architecture and reviews; Crog implements. If Crog disagrees with an architectural decision, it raises the concern in the PR description or asks [PO NAME] to relay it; it does not silently deviate.

### Starting a New Clead Chat

Clead has no persistent memory between conversations. When opening a new chat session, run `tools/dump.sh` from the project root in Git Bash and paste the output file's contents into the chat. This gives Clead immediate full context — current code, documentation, and git history.

```bash
bash tools/dump.sh
```

Paste the contents of the generated `.txt` file as the first message in the new chat. Clead will be fully oriented and ready to continue without any re-briefing.

---

## 5. Code Review Process

All work merges to `main` via Pull Request. Direct pushes to `main` are not permitted.

### Branch Protection

`main` is protected by a GitHub ruleset (enforced under the GitHub Team plan). The following rules are active:

| Rule | Setting | Effect |
|---|---|---|
| Restrict deletions | On | `main` cannot be deleted |
| Require a pull request before merging | On — 0 required approvals | All changes must go through a PR; no direct pushes |
| Require status checks to pass | On — `build` job required | CI (lint, format, tests, coverage) must be green before merge |
| Require branches to be up to date | On | PR branch must include latest `main` before it can pass CI and merge |
| Block force pushes | On | History on `main` cannot be rewritten |

These rules are enforced at the GitHub level — not a convention, not a guideline. A PR that fails CI cannot be merged regardless of who is trying. CI is the hard gate; 0 formal approvals are required.

**Bypass:** The bypass list is empty. As repo owner, [PO NAME] retains admin-level override capability at the GitHub account level if genuinely needed.

### Reviewer Roles and Access

| Reviewer | GitHub access | Review method |
|---|---|---|
| Clead | None — no direct GitHub or repo access | [PO NAME] runs `tools/pr_dump.sh` → pastes output into chat; Clead provides feedback; [PO NAME] posts it as a PR comment. **Required on every PR.** |
| Copi (GitHub Copilot) | Full — native GitHub integration | Reviews PRs directly in GitHub via Copilot code review. **Optional — on request if licence is available.** |
| [OPTIONAL: human reviewer] | Full — direct GitHub access as collaborator | Reviews PRs directly in GitHub; leaves inline comments. Available async. |

### Clead Review Standard

Every PR reviewed by Clead must include all of the following, without being asked:

**1. Threat model statement**
Before the review findings, state what threat model is being applied. Example:
> "Reviewing this as a file parser — primary risks are framing corruption, silent data loss, and type coercion edge cases."

**2. TPS compliance check**
Every code PR must be checked against `docs/TECHNICAL_PRODUCT_SPECIFICATION.md`. Explicitly confirm or flag:
- Does the implementation match what the TPS says this component must do?
- Does the public interface (function signatures, return types, error behaviour) match the TPS contract?

**3. Focused second pass on error handling and type validation**
After the general architectural review, always do an explicit second pass covering:
- Every validation function: are bool inputs rejected where int/float is expected?
- Every error path: are exceptions caught at the right level?
- Every external input: is it validated before use?
- Any type coercion that could silently accept unexpected values?

**4. Test quality check**
Coverage percentage and test count are necessary but not sufficient. Every code PR must include a test coverage narrative table in the PR description (see `docs/CROG_ONBOARDING.md` section 3). Clead must verify:
- Does the table exist? If not, request it before approving.
- For each row in the table: does the named test actually assert what the table claims?
- Are all non-trivial error paths and recovery behaviours represented in the table?
- For every error path in the implementation, is there a test that verifies the correct recovery behaviour (not just that a warning was logged)?

**5. What I did not check**
Every approval must end with an explicit list of what was not verified. Example:
> "Did not check: interaction with other modules not in this diff; behaviour under concurrent access."

These requirements exist because Clead reviews from the diff only (not the full file), which creates structural blind spots. Copi reads full files and catches cross-section inconsistencies — Clead's role is architectural alignment, spec compliance, and test quality. Together they cover different failure modes.

**6. Verdict prompt discipline**
Clead's verdict is delivered as a single Crog prompt with no preamble or chat commentary. The review summary goes into the PR comment via Crog — not into the chat. The prompt block must be the only content in Clead's post so [PO NAME] can copy-paste it directly.

**7. Copi review gate**
Clead's verdict prompt includes a merge instruction if and only if Copi has completed its review and has no open comments requiring resolution. If Copi has not yet reviewed, or has open comments, the verdict prompt must not include a merge instruction — the merge prompt is issued separately once Copi is satisfied. This gate applies to code PRs only. Docs/tooling PRs do not require Copi review and may be merged on Clead's approval alone.

---

### Direction A — Feature/Fix PR (code)

1. Crog opens PR from `feature/<name>` or `fix/<name>` to `main`
2. Copi review auto-requested by workflow
3. Crog polls until Copi completes, waits 10s, posts pr_dump as PR comment
4. Crog reports PR URL to [PO NAME]
5. [PO NAME] drops URL into Clead's chat
6. Clead fetches PR directly, reads diff + Copi comments + pr_dump
7. If changes needed: Clead produces fix prompt → [PO NAME] pastes → Crog pushes → go to step 3
8. If approved: Clead produces verdict + merge prompt → [PO NAME] pastes → Crog posts comment and merges

---

### Direction B — Docs/Tooling PR (Copi is not involved — go straight to Clead)

1. Crog opens PR from `docs/<name>` or `tooling/<name>` to `main`
2. Crog posts pr_dump as PR comment: `gh pr comment <PR-number> --body "$(bash tools/pr_dump.sh <PR-number> --no-src)"` and reports PR URL to [PO NAME] with `?i=1` (increment `i` by 1 on each re-report of the same PR)
3. [PO NAME] drops URL into Clead's chat
4. Clead fetches PR directly, reads diff + pr_dump
5. If Clead requests changes: Clead produces fix prompt → [PO NAME] pastes → Crog implements only what the prompt specifies → pushes → posts pr_dump → reports `?i=1` to [PO NAME] (increment `i` by 1 on each re-report of the same PR) → **stops and waits**. Go back to step 3.
6. If approved: Clead produces verdict + merge prompt → [PO NAME] pastes → Crog posts comment and merges

**Hard stop rule:** After posting the pr_dump and reporting back to [PO NAME], Crog stops completely — for both PR types. For code PRs, this follows Copi's review; for docs/tooling PRs, Copi is not involved and the stop applies immediately after posting pr_dump. In both cases: Crog does not read or act on Copi's comments, does not push any fix based on Copi's findings, and waits for [PO NAME] to paste Clead's instruction. Clead is the mandatory gate on every iteration. No exceptions except Crog's own unambiguous mechanical mistakes before the first Clead review.

---

### Copi suspended mode

When Adam announces "Copi suspended", all PR reviews run without Copi until Adam announces resumption.

**Crog behaviour:**
- Skip `copi_wait.sh` and Copi review request entirely.
- After opening PR and posting pr_dump, report PR URL and changed file URLs to Adam (file URL reporting applies to all PR reports — see `docs/CROG_ONBOARDING.md`).
- Clead reviews with full file context as sole reviewer.
- Fix loop continues as normal (Clead ↔ Crog) until Clead approves.

**Clead behaviour:**
- Reviews full file content (not diff-only) for each changed file.
- Flags if additional files outside the PR are needed for cross-module review.
- Issues verdict and merge prompt as normal.

On resumption, standard Copi flow resumes immediately.

---

### Decision Authority on PRs

| Decision type | Final authority |
|---|---|
| Merge approval on Crog's code | Clead |
| Merge approval on Clead's code | Crog (with Clead retaining final technical call) |
| Merge approval on reviewer's code | Clead |
| Override any technical decision | [PO NAME] (PO) |

---

## 6. Architectural Review & Feedback Loop

The review process is bidirectional — not just Crog's code being reviewed by Clead, but Clead's architectural proposals being reviewed by Crog for practical feasibility.

### Clead → Crog (Architecture review)

1. Clead produces an architectural proposal, design document, or specification
2. [PO NAME] hands the task to Crog in the terminal session
3. Crog reviews from an implementation perspective — feasibility, edge cases, gotchas, fit with the tech stack
4. Crog flags concerns or suggests alternatives in the PR description or asks [PO NAME] to relay them to Clead
5. Clead considers the feedback, updates the design if warranted, and makes the final architectural call

### Crog → Clead (Code review)

1. Crog implements based on approved architecture, opens a PR
2. [PO NAME] runs `tools/pr_dump.sh` and pastes the output into Clead's chat
3. Clead reviews for architectural alignment, code quality, and correctness
4. Clead approves or requests changes with specific, actionable feedback

---

## 7. Decision Authority

| Decision type | Final authority |
|---|---|
| Product direction, features, scope | [PO NAME] |
| Technical architecture, design patterns, tooling | Clead |
| Implementation approach, CLI-level choices | Crog (with Clead oversight) |

Clead's architectural decisions can be challenged by Crog through the feedback loop, but Clead has the final technical call. [PO NAME] can override any technical decision if needed, but aims to stay hands-off on technical matters in practice.

---

## 8. Secondary Environment

When working from a secondary machine (e.g., a laptop or field kit):

1. All analysis, code changes, and documentation happen on [PRIMARY MACHINE]
2. If light coding is needed on [SECONDARY MACHINE], Crog can be invoked there — Claude Code is installed for that reason
3. Code written on [SECONDARY MACHINE] is committed and pushed to GitHub, then pulled on [PRIMARY MACHINE] for review

---

## 9. Best Practices

**Keep diffs focused.** Only paste changed files or diffs to Clead — not the entire codebase. Use `git diff --name-only` first to identify what changed, then paste only the relevant files (or use `tools/pr_dump.sh` for PR review).

**One task at a time.** Complete a task, review it, commit it, then move to the next. Don't accumulate unreviewed changes across multiple features.

**GitHub is the source of truth.** Every agreed decision that affects code or architecture ends in a commit. If it is not in the repo, it does not exist.

**Commit messages are communication.** Write them for the team, not just for the log. Imperative tense, present tense, specific — `Add configurable UDP listener with multicast support`, not `stuff`.

**Start new Clead chats with a repo dump.** Run `bash tools/dump.sh` and paste the output at the start of any new Clead session. This is faster and more reliable than relying on memory.

**Crog speaks up.** Crog is not a passive code generator. If something looks wrong, contradicts the TPS, or has a missing test case — it raises it. See `docs/CROG_ONBOARDING.md` for the full mandate.

---

## 10. Summary

[PO NAME] owns the product. Clead owns the architecture and quality. Crog owns the implementation. They work together on [PRIMARY MACHINE] — Clead in the browser, Crog in the terminal — with no cross-machine relay, no synchronisation overhead. GitHub is the shared memory. Git diffs are the language they all speak. CI on `main` is the hard gate — nothing merges without lint, format, tests, and coverage passing. Clead reviews every PR via pr_dump; Copi reviews on request; Crog merges on [PO NAME]'s explicit instruction.
