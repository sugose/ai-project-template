# Next Session — Workflow Automation Study

**Date:** 2026-06-27
**Status:** Open — decision required before implementation
**Owner:** Adam + Clead

---

## Context

Adam has commissioned a study on eliminating his own involvement from workflow steps that (a) can be replaced by machine and (b) add no value. This document captures the current state of that analysis and the decisions needed to proceed.

---

## The Problem

Almost all of Adam's copy-pasting in the current workflow is relay work, not governance work. The `?i=1` convention, the "STOP and wait" rule, the exact report format, the paste of fix prompts — all of this scaffolding exists only because Clead (Claude Chat) and Crog (Claude Code CLI) cannot talk to each other directly. It is not a process control. It is a workaround.

---

## Adam's Current Touchpoints — Scored

| # | What Adam does | Replaceable? | Adds value? | Verdict |
|---|---|---|---|---|
| 1 | Picks next PBI from backlog | No — product judgment | Yes | **Keep** |
| 2 | Pastes Crog task prompt into Claude Code | Yes | No | **Eliminate** |
| 3 | Drops PR URL into Clead's chat | Yes | No | **Eliminate** |
| 4 | Pastes Clead's fix prompt into Crog | Yes | No | **Eliminate** |
| 5 | Posts Clead's verdict as PR comment | Yes | No | **Eliminate** |
| 6 | Merges approved PR | Yes technically | Yes — hard control | **Keep** |
| 7 | Runs dump.sh and pastes into new Clead session | Yes | No | **Eliminate** |
| 8 | Updates CHANGELOG.md after merge | Yes | No | **Eliminate** |
| 9 | Substitutes Crog token in curl command | Yes (blocked on Anthropic secrets support) | No | **Eliminate when unblocked** |

After elimination, Adam's role collapses to: **pick the next PBI, approve the merge, override if something looks wrong.**

---

## What Has to Change

Three things, in dependency order:

**1. Clead gets a GitHub write channel**
Currently Clead has no GitHub access. Clead needs to post PR comments (verdicts, fix requests) directly rather than routing through Adam → clipboard → Crog. Cleanest path: Crog acts as Clead's hands for GitHub writes, receiving instructions via a shared artifact (PR comment, repo file, or webhook) rather than Adam's clipboard.

**2. Crog gets a trigger mechanism that doesn't require Adam**
Currently Crog only runs when Adam opens a Claude Code session and pastes a prompt. With Routines (documented in `docs/ROUTINES.md`) and eventually Anthropic secrets support, Clead fires Crog directly via curl. That closes the loop.

**3. Session startup becomes a trigger, not a ritual**
`dump.sh` + paste is currently how Clead cold-starts. Once Clead runs as a Routine with the repo attached, it reads state from GitHub directly. The dump.sh mechanism becomes redundant for Routines-based sessions (stays useful as a fallback for chat-based sessions).

---

## The Surface Question

Moving to full automation requires Clead to move from Claude Chat to a Routine or Cowork context. This is not a swap — it is an addition of a second mode:

| Mode | Surface | When to use |
|---|---|---|
| **Chat mode** (current) | Claude Chat | Planning, architecture, complex review, this kind of conversation |
| **Routine mode** (proposed) | Claude Routine / Cowork | Pure execution: PBI → implement → review → merge, no Adam in the middle |

The two modes coexist. Chat mode stays exactly as is. Routine mode handles execution loops autonomously.

---

## The One Risk That Must Be Protected

**Reviewer independence.** Clead's review quality depends on seeing only the diff and the pr_dump — not the full implementation context that Crog had. If Clead and Crog share a session context, Clead stops being an independent reviewer and becomes a rubber stamp.

**Mitigation:** In Routine mode, Clead's review subagent must be initialized with only the pr_dump output and the TPS. Nothing else. This discipline must be structural — built into the Routine design, not assumed.

---

## Recommended First Step (Experiment Before Commitment)

Before redesigning anything, run one experiment on a real PR:

1. Crog posts the PR URL to a known location (a PR comment with a marker tag)
2. Clead polls for it, reviews using only the pr_dump
3. Clead posts the fix prompt or verdict back as a PR comment with a marker tag
4. Crog picks it up on its next check-in

No new infrastructure. Just wires what already exists in `docs/ROUTINES.md`. If it works cleanly on one PR, we have proof of concept. If it breaks, we know exactly where before committing to a full redesign.

---

## Decisions Needed at Next Session

1. **Approve the experiment** — run one PR through the Crog-posts / Clead-polls / Crog-picks-up loop. Which repo and which PBI should be the test case?
2. **Confirm the two-mode model** — Chat for thinking, Routine for execution. Any objections or refinements?
3. **Reviewer independence design** — agree on exactly what inputs Clead's review subagent receives and what it is explicitly denied.
4. **Scope boundary** — does this change land in the template repo first (so all projects inherit it), or in one project repo as a pilot?

---

## What This Is Not

This is not a proposal to automate merge authority. Adam merges. That is the one genuine human control in the loop and it stays.

This is not a proposal to remove Clead's review gate. Every PR still goes through Clead. The review standard (threat model, TPS compliance, error handling pass, test quality check, what I did not check) is unchanged.

This is not urgent. The current workflow functions. This is an improvement to be landed carefully, not a fire to be fought.
