# Controlling the Uncontrollable: Lessons from Building a Production AI Development Workflow

*A case study in GitHub Copilot integration, multi-agent coordination, and the engineering discipline required to make AI-assisted development actually work.*

---

## Introduction

I did not set out to build a workflow. I set out to ship a Python blackjack simulation.

My background is agile — coach, scrum master, team lead, product owner, product manager, across industries and teams for more years than I care to count. I started programming back in the days when each line of code was pre-pended by a number, preferably divisible by ten. So I have opinions about software development that predate most of the tools discussed in this article.

I also have a leading star. It goes like this: my leading star doesn't tell me what we should do next to maximise customer value. It tells me what I can do today so the team I'm working with actually functions well. I mention it here because it is relevant — but not in the way you might expect. That connection will become clearer by the end.

What I ended up building — almost by accident — was a structured, multi-agent development system with explicit review gates, a label-based automation model, and a set of hard-won process rules that now govern every line of code that goes into the project. This article describes what we built, why we built it, and specifically what we learned about using GitHub Copilot as part of a serious development workflow rather than a convenience tool.

The team is small by design: a product owner who holds authority over scope and direction, an AI tech lead (Claude, operating in a chat interface) responsible for architecture and code review, an AI developer (Claude Code CLI) responsible for implementation, and GitHub Copilot Business as an automated code reviewer. The human is the only one who merges. The AIs do everything else.

This is not a story about AI replacing developers. It is a story about what happens when you try to run AI agents like a professional engineering team — with specifications, review gates, process rules, and the expectation that things will go wrong in ways you did not anticipate.

---

## The First Problem: One Review, Then Silence

The most immediate Copilot integration issue was not what I expected.

The GitHub Copilot code review feature works well the first time. Open a pull request, and Copilot reviews it automatically via a repository ruleset. The findings are useful: style violations, potential bugs, test coverage gaps, cross-file inconsistencies that the diff makes visible. On a multi-iteration workflow — where a PR might go through three or four fix rounds before merge — the first review is valuable.

The second review is the problem.

With "Review new pushes" enabled in the ruleset, Copilot is supposed to re-review when new commits are pushed to an open PR. In practice, it does not. Community reports suggest it fails to re-review on roughly one in three pushes. In our workflow, where every PR iterates at least twice, this was not a one-in-three problem — it was a consistent problem. Copilot would review at PR open and then, on subsequent pushes, produce nothing.

We tried everything documented. The GitHub REST API (`POST /pulls/{n}/requested_reviewers`) works for the initial review request but returns 422 after Copilot has already reviewed once. The GraphQL `requestReviews` mutation behaves the same way. The `gh pr edit --add-reviewer @copilot` CLI command: same result. There is no programmatic re-review path. The only reliable method is clicking "Re-request review" in the GitHub UI — which is not acceptable in an automated workflow.

We investigated the failure mode carefully. What we observed over multiple PRs was this: a push containing only documentation changes (no source files) between two source code pushes consistently triggered re-review on the subsequent source push, even when a direct source-to-source push did not. Our hypothesis is that Copilot maintains an internal baseline of its last reviewed commit, and uses this to deduplicate. A documentation push shifts the baseline without triggering a review, so the next source push arrives as a genuinely new diff — unreviewed content — and Copilot fires.

We documented this finding and posted it to the GitHub community forum. We have two consistent data points. It is not yet confirmed as a reliable pattern.

Rather than continue chasing the re-review problem, we redesigned the trigger model entirely.

---

## The Solution: Label-Based, Interrupt-Driven Review

The core insight from the re-review investigation was this: Copilot was being used as an event-driven tool (run on every push) when it should be used as an interrupt-driven tool (run when a human decides it adds value).

The new model uses a GitHub Actions workflow triggered by a label:

```yaml
name: Copilot PR Review (on label)

on:
  pull_request:
    types: [labeled]

jobs:
  copilot-review:
    if: ${{ github.event.label.name == 'ai-review' }}
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write

    concurrency:
      group: ai-review-${{ github.event.pull_request.number }}
      cancel-in-progress: false

    steps:
      - name: Request Copilot review
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: gh pr edit ${{ github.event.pull_request.number }} --add-reviewer @copilot --repo ${{ github.repository }}

      - name: Remove ai-review label
        if: always()
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: gh pr edit ${{ github.event.pull_request.number }} --remove-label ai-review --repo ${{ github.repository }}
```

Adding the `ai-review` label triggers exactly one Copilot review. The label is automatically removed after every run — including failed runs — preventing ghost state and accidental re-triggers. A `concurrency` block prevents parallel runs on the same PR. The "Review new pushes" ruleset setting is disabled.

The label lifecycle is governed by explicit invariants:

- The label must not be added if already present
- The label must not be added while a run is already in progress
- The label must not be added unless a new run is expected to produce meaningfully different input
- At most one Copilot review per review phase (initial or pre-merge)
- Re-review is permitted only after changes affecting program behaviour — logic, interfaces, or data flow. Formatting changes do not justify re-review.
- If re-review fails, do not retry. Escalate to full-file human review instead.

This approach reduced Copilot credit consumption by an estimated 70-80% compared to the "Review new pushes" model, while increasing the signal-to-noise ratio of each review. Instead of five automated passes on a PR that iterates five times, we get one or two deliberate passes at moments where the review will actually change the outcome.

The ruleset with "Review new pushes" is event-driven automation. The label model is interrupt-driven control. That distinction matters more than it appears.

---

## The Second Problem: Copilot Is Not a Reviewer

Running Copilot as a label-triggered tool forced a more honest accounting of what it actually does.

Copilot reviews the PR diff. It does not review the codebase. It cannot detect inconsistencies in code that is not part of the diff, even if the change logically affects it. This is a fundamental constraint of diff-based review, and it has real consequences.

On one PR in this project — a strategy interface change — Copilot correctly reviewed the modified file and found nothing wrong. It did not flag that a dependent file, not included in the diff, still called the old interface with the old signature. The bug would have shipped had the tech lead not performed a cross-file consistency check. Copilot's review was accurate within its scope. Its scope was too narrow.

The correct mental model for Copilot is not "reviewer" but "heuristic analyser." It provides fast, targeted, advisory input on visible diff content. Its output is never authoritative. It catches what it can see. For what it cannot see — interface contracts, implicit dependencies, cross-file consistency — a human (or an AI with full file context) must do the work.

This led to a formal review depth policy for the tech lead role:

**Structured dump only** — for documentation and tooling PRs with low risk; small single-file changes with clear bounded scope.

**Changed files fetched** — for code PRs with clear scope; when the PR dump raises a question that needs file context to answer.

**Full file fetch including potentially affected files** — for interface changes; multi-file PRs; any PR touching critical paths; any PR where cross-file consistency is in question.

The tech lead states the review depth applied and the reason in every verdict. This creates a lightweight audit trail and forces the reviewer to be explicit about what was and was not checked.

When Copilot's credits are exhausted or the tool is unavailable, the tech lead operates as sole reviewer with elevated depth. The process does not change. The coverage does not degrade. Copilot's absence is an inconvenience, not a failure mode.

---

## The Third Problem: Multi-Agent Coordination Without Chaos

Running AI agents as a development team introduces coordination problems that do not exist in human teams — or rather, it exposes coordination problems that human teams solve informally, through conversation and shared context, in ways that AI agents cannot.

The most important rule we developed is the hard stop.

After the developer agent opens a PR and posts a structured dump of the PR state as a comment, it stops completely. It does not read the reviewer's comments. It does not push fixes. It does not act on anything it observes in the PR. It waits.

Every iteration is gated by the tech lead. The developer acts only on explicit prompts. Nothing autonomous happens between the dump and the verdict.

This rule exists because we learned what happens without it. On an early PR — before the rule was formalised — the developer agent observed Copilot's comments, attempted to interpret them, and pushed what it thought were fixes. The fixes were incorrect. They introduced a subtle spec change that was not requested. The spec drift was invisible until three PRs later, when something downstream broke. Unwinding it cost more time than the original fix would have taken.

The hard stop rule means that autonomy is earned, not assumed. An AI agent that acts on its own interpretation of reviewer comments — without human oversight of that interpretation — introduces spec drift at a rate that compounds silently. The constraint feels slow. The alternative is slower.

A second coordination rule is the prompt structure: every instruction to the developer agent uses clearly delimited START PROMPT / END PROMPT markers and contains exactly one task. Combining tasks in a single prompt produces ambiguous results. The developer agent is good at executing precise instructions and unreliable at prioritising between competing ones.

The third rule is session hygiene: every session ends with a housekeeping PR that updates the working memory document, the backlog, and the changelog. Context does not persist between AI sessions. The documents are the continuity mechanism. When the documents are stale, the next session starts from a degraded state. We learned to treat the housekeeping PR with the same rigour as a feature PR.

---

## The Team Structure and Its Invariants

The team structure has four roles with explicit authority boundaries:

**Product owner** — the human. Decides what gets built. Approves merges. Owns label lifecycle management. The final word on scope.

**Tech lead (AI)** — architectural authority. Reviews all PRs. Decides when Copilot review adds value. Produces verdicts and fix prompts. Never merges unilaterally.

**Developer (AI)** — implementation only. Opens PRs, posts dumps, waits for instruction. Never acts on reviewer findings without an explicit prompt. Never pushes between dump and verdict.

**Copilot** — heuristic analyser. Reviews diffs on request. Output is advisory. Cannot satisfy required-reviewer branch protection rules.

The model works because each role has a lane and does not drift into another's. The tech lead does not implement. The developer does not review. The product owner does not write prompts. Copilot does not gate merges.

Role drift — an AI agent acting outside its defined scope — is the failure mode most difficult to detect. It looks like progress. The agent is doing things. The things appear plausible. The spec is quietly diverging from what was decided. The detection mechanism is the hard stop rule and the consistency check that opens every session: the tech lead reads all living documents and flags any inconsistencies before work begins.

---

## What the Process Produced

After approximately fifteen sessions over several months, the project has 247 tests, 98.8% coverage, a fully specified event model, a human-playable CLI session, and a process that can be handed to a new AI agent at session start with a single dump command.

More usefully, it produced a set of documented failure modes:

**Spec drift under autonomous iteration** — the PR #58 incident. Seven iterations between the developer and Copilot without tech lead involvement. Incorrect spec change shipped. Hard stop rule created as a result.

**Diff-bounded review missing cross-file bugs** — the strategy interface incident. Copilot reviewed correctly within its scope. Dependent file not in diff. Bug caught by tech lead cross-file check. Full-file review policy created as a result.

**Re-review credit drain** — "Review new pushes" causing 5× the necessary reviews on a multi-iteration PR. Label-based model created as a result.

**Tool fragility** — `copi_wait.sh` polling script timing out before Copilot completed. Script eventually deleted when label model made it unnecessary.

Each failure produced a process change. Each process change was documented in a PR. The process is itself a product, versioned and maintained with the same rigour as the source code.

---

## A Note on Teams Without Claude

The model described here uses Claude as the tech lead. For teams operating under policies that restrict third-party AI services, the tech lead role can be filled by a human reviewer using GitHub Copilot Chat as a reasoning aid.

The label-based Copilot review model (Layer 1) and the process layer (Layer 3) are unchanged. What changes is Layer 2: the human tech lead uses Copilot Chat to ask questions the diff cannot answer.

For interface changes: *"Which files in this repo call `[function name]`? Are there any that aren't in this PR's diff?"*

For multi-file PRs: *"Given these changes to `[file A]`, are there any inconsistencies with `[file B]`?"*

For unfamiliar code paths: *"Explain what `[function]` does and what would break if its signature changed."*

Copilot Chat has full repository context via workspace indexing, though it may not have complete coverage of all files. It cannot issue a formal verdict — that remains the human's decision. It can accelerate the cross-file consistency check that diff-based automated review cannot perform.

The process invariants are identical. The enforcement model is identical. The only thing that changes is who sits in Layer 2.

---

## Conclusion

What we built is not a Copilot workflow. It is a controlled execution model for a non-deterministic AI service.

The distinction matters. A workflow is a set of steps. A controlled execution model is a set of invariants that constrain behaviour under failure, under ambiguity, and under pressure. The invariants — hard stop, label lifecycle, review depth policy, single-task prompts, session hygiene — exist not because the happy path needs them, but because the failure modes do.

The engineering problems we solved — re-review reliability, credit consumption, diff-bounded review quality, multi-agent coordination — are not GitHub problems or Copilot problems. They are integration problems. They arise wherever a non-deterministic, stateless, cost-incurring tool is embedded in a process that requires deterministic, stateful, cost-controlled behaviour.

The solutions we found are transferable. The label model works for any team. The review depth policy works for any tech lead. The hard stop rule works for any developer agent. The session hygiene works for any context-limited AI.

What is not transferable is the learning. The process we have now is the result of the process failing in specific ways and being fixed in specific PRs. That history is in the changelog. It is in the onboarding documents. It is in the cautionary cases cited in the team structure documentation.

The teams that will use AI well are the teams that treat AI failures as process failures — and fix them accordingly.

---

*The project described in this article is open source: [github.com/sugose/python-blackjack](https://github.com/sugose/python-blackjack). The template repository capturing the workflow and tooling patterns is available at [github.com/sugose/ai-project-template](https://github.com/sugose/ai-project-template).*
