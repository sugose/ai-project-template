# GitHub Copilot Code Review — Limitations & Mitigations

*Based on operational experience with Copilot Business on a multi-iteration AI-assisted development workflow. Updated June 2026.*

---

## 1. One review per PR by default

**Limitation:** With the native ruleset ("Automatically request Copilot code review"), Copilot reviews a PR once on open and never again, unless "Review new pushes" is also enabled.

**Mitigation:** Enable "Review new pushes" in the ruleset, or use a label-based workflow to trigger reviews explicitly. See section 3 for why "Review new pushes" has its own problems.

**Acceptance:** For simple PRs that don't iterate, one review is sufficient.

---

## 2. No programmatic re-review after initial review

**Limitation:** Once Copilot has submitted a review on a PR, there is no API or CLI method to trigger a re-review. `gh pr edit --add-reviewer @copilot` and `POST /pulls/{n}/requested_reviewers` both work for the initial request but silently fail or 422 after the first review. The only guaranteed re-review path is clicking "Re-request review" in the GitHub UI.

**Mitigation:** Under the label-based workflow, removing and re-adding the `ai-review` label may trigger a new workflow run. Under the ruleset approach, "Review new pushes" is supposed to handle this automatically but is unreliable (see section 3).

**Acceptance:** For now, manual UI re-request remains the fallback. GitHub community has an open feature request for API support (https://github.com/orgs/community/discussions/186152).

---

## 3. "Review new pushes" is unreliable

**Limitation:** Even with "Review new pushes" enabled in the ruleset, Copilot frequently does not re-review on subsequent pushes to an open PR. Community reports suggest roughly a 1-in-3 failure rate. The cause is opaque — Copilot appears to use an internal deduplication mechanism based on the diff since its last review, and suppresses re-reviews when it determines nothing meaningful has changed.

**Observed behaviour:** In our workflow, straight code→code pushes consistently failed to trigger re-review. However, a docs-only push followed by a code push reliably triggered re-review in two separate observations.

**Hypothesis:** The docs push shifts Copilot's internal baseline commit, causing the subsequent code push to be seen as a fresh diff. Unconfirmed — two data points only. Posted to GitHub community for validation: https://github.com/orgs/community/discussions/186152

**Mitigation:** Moved to a label-based workflow that bypasses "Review new pushes" entirely. Review is triggered explicitly when needed rather than automatically on every push.

**Acceptance:** The hypothesis remains unproven. The label-based approach sidesteps the problem rather than solving it.

---

## 4. Token/credit consumption with "Review new pushes" enabled

**Limitation:** "Review new pushes" causes Copilot to run a full review on every push to every open PR. On a multi-iteration workflow (5+ pushes per PR, 10+ files, moderate diff size), this amounts to 5× the reviews actually needed. Each review consumes AI credits, and from June 1 2026, also consumes GitHub Actions minutes on private repositories.

**Mitigation:** Disable "Review new pushes." Use label-based triggering to run Copilot exactly once per PR, or at specific deliberate points.

**Acceptance:** None — this is avoidable waste. Label-based is strictly better for credit management.

---

## 5. No filtering by file path in the ruleset

**Limitation:** The native ruleset trigger runs Copilot on all PRs matching the branch condition, regardless of which files changed. There is no built-in way to say "only run Copilot if src/ files were changed."

**Mitigation:** Under the label-based approach, the tech lead makes the file-scope decision manually — no label is added for docs-only PRs.

**Acceptance:** The label-based approach solves this more elegantly than path-scoped workflow triggers.

---

## 6. Review quality is diff-bounded

**Limitation:** Copilot reviews the PR diff, not the full codebase. Cross-file consistency issues that don't appear in the diff are missed. Example: if a function signature changes in one file and another file also calls that function but isn't in the diff, Copilot will not flag the inconsistency.

**Observed failure:** A strategy interface change in one file was not caught in a dependent file on the initial review pass — Copilot only flagged it when the dependent file appeared in a later iteration's diff. Had the PR been merged after the first iteration, the bug would have shipped.

**Mitigation:** The tech lead performs full-file reviews (fetching all changed files plus potentially affected files) for interface changes and multi-file PRs. This compensates for Copilot's diff-bounded view.

**Acceptance:** Accepted as a fundamental limitation of diff-based review. Human/AI review with full context remains necessary for interface changes.

---

## 7. Detection latency and polling unreliability

**Limitation:** Copilot review completion is not synchronised to any webhook or API event that is easily pollable. Polling scripts frequently time out before Copilot finishes, even with generous timeouts.

**Mitigation:** Under the label-based workflow, polling scripts are no longer needed. The product owner confirms Copilot is done by checking the GitHub UI before handing the PR to the tech lead for review.

**Acceptance:** Acceptable under the new flow — the human check replaces the unreliable poll.

---

## 8. Review does not satisfy branch protection required reviewers

**Limitation:** Copilot always leaves a "Comment" review, never "Approve" or "Request changes." It cannot satisfy a CODEOWNERS requirement or a "required reviewers" branch protection rule. Merge is never blocked by Copilot's review alone.

**Mitigation:** The tech lead issues the explicit merge authorisation. The developer merges on the tech lead's instruction, not on Copilot's review state.

**Acceptance:** This is by design — the tech lead is the merge authority, not Copilot.

---

## Our Current Review Setup

This section describes the review policies and workflow we have arrived at after extensive operational experience with Copilot Business in a multi-iteration AI-assisted development workflow.

### Team structure

The team consists of a product owner, a tech lead (AI), a developer (AI), and Copilot as an automated code reviewer. The product owner coordinates between roles. The tech lead owns architecture, design decisions, and PR review authority. The developer implements and pushes code. Copilot provides an automated first-pass review on code PRs when explicitly requested.

### PR flow

1. The developer opens a PR and posts a structured dump of the PR state as a comment.
2. The product owner hands the PR URL to the tech lead.
3. The tech lead reviews the PR — see review depth policy below — and issues a verdict.
4. If changes are needed, the developer pushes a fix and the process repeats.
5. When the tech lead approves, the developer merges.

### Review depth policy

The tech lead determines review depth dynamically per PR:

- **Structured dump only** — docs and tooling PRs with low risk; small single-file changes with clear bounded scope.
- **Changed files fetched** — code PRs with clear scope; when the dump raises a question needing file context.
- **Full file fetch including potentially affected files** — interface changes; multi-file PRs; anything touching critical paths; any PR where cross-file consistency is in question.

The tech lead states the review depth applied and the reason in every verdict.

### Copilot review policy

Copilot is invoked via a label-based GitHub Actions workflow (`.github/workflows/copilot-review.yml`). Adding the `ai-review` label to a PR triggers a single Copilot review. The tech lead decides when to request Copilot review and includes the instruction in the verdict.

**Copilot review is requested for:**
- Large or risky code changes spanning multiple source files
- New modules or significant interface changes
- Final pre-merge sanity check on complex PRs
- When the tech lead's own review flags uncertainty

**Copilot review is not requested for:**
- Docs and tooling-only PRs
- Small targeted fixes
- Iterative commits on a PR that already had a Copilot pass

### Re-review policy

For a Copilot re-review after a fix push, the product owner removes and re-adds the `ai-review` label. Whether this reliably triggers a new review is an open question — see limitation 2. If it fails, the tech lead's full-file review serves as the quality gate.

### Credit management

Copilot credits are finite. The label-based approach ensures Copilot runs only when the tech lead judges the cost justified. Credit exhaustion suspends Copilot review entirely; the tech lead operates as sole reviewer with elevated review depth until credits are restored.

---

## Summary

| Limitation | Severity | Mitigation | Status |
|---|---|---|---|
| One review per PR by default | Medium | Label-based trigger | ✅ Resolved |
| No programmatic re-review | High | Label remove/re-add; manual UI fallback | ⚠️ Partial |
| "Review new pushes" unreliable | High | Label-based replaces ruleset trigger | ✅ Resolved |
| Token drain with review_on_push | High | Disabled review_on_push | ✅ Resolved |
| No file-path filtering in ruleset | Low | Tech lead decides label per PR type | ✅ Resolved |
| Diff-bounded review quality | Medium | Tech lead full-file review for interface/multi-file PRs | ✅ Mitigated |
| Detection latency / polling | Medium | Polling removed; human UI check | ✅ Resolved |
| Cannot satisfy required reviewers | Low | Tech lead is merge authority | ✅ Accepted |

---

*"I told you it wouldn't work." — Paco, CEO*
