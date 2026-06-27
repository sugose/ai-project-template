# Proposal: Rebuilt AI-Assisted Development Workflow
**Date:** 2026-06-27
**Status:** Proposal — decision required
**Author:** Clead (Claude, acting as Tech Owner)
**Context:** Written at end of session after evaluating current workflow complexity,
available tooling, and the principle that Adam should only be involved where
(a) a machine cannot do what he does, or (b) he adds genuine value.

---

## 1. The Problem With What We Built

The current workflow works. But it is more complex than the problem requires.

Almost every piece of custom scaffolding in the current template —
`dump.sh`, `pr_dump.sh`, the `?i=1` convention, the hard stop rule,
the STOP-and-wait pattern, the NEXT_SESSION files, the dump-and-paste
session startup ritual — exists for the same reason: Clead (Claude Chat)
and Crog (Claude Code CLI) could not talk to each other, and Clead had
no GitHub access. Adam's clipboard was the only channel between them.

That constraint no longer exists. The scaffolding that compensated for it
should not be carried forward into a rebuild.

A second problem: the workflow was discovered by running into walls. Every
rule in TEAM_STRUCTURE.md, every convention in CROG_ONBOARDING.md, every
tool in the `tools/` directory represents a problem that was encountered,
diagnosed, and patched. The result is a system that works but whose
complexity is proportional to the number of walls hit, not to the
complexity of the underlying problem.

Starting over, informed by what we learned, using tools that now exist,
produces something much smaller.

---

## 2. What We Know Works (Keep)

These are not workarounds. They are genuine process controls that belong
in any rebuild:

- **Role separation**: one agent implements, a separate agent reviews.
  The reviewer must have a different context from the implementer —
  not just a different prompt, but structurally isolated inputs.
- **TDD discipline**: tests first (red), implementation (green), refactor.
  Non-negotiable. Catches real bugs.
- **Branch protection + CI as the hard gate**: nothing merges without
  lint, format, tests, and coverage passing. Enforced at GitHub level.
- **Adam as sole merge authority**: the one genuine human control.
  Not friction — a decision point that requires domain judgment.
- **The Clead Review Standard**: threat model statement, TPS compliance
  check, error handling second pass, test quality check, explicit list
  of what was not checked. This is the right review framework. Keep it
  verbatim.
- **GitHub as single source of truth**: if it is not in the repo,
  it does not exist.
- **One task at a time**: complete, review, merge, then next. No
  accumulating unreviewed changes.
- **The spec precedes the implementation**: Clead produces or validates
  a spec before Crog writes a line of code.

---

## 3. What We Know Doesn't Work (Discard)

- **dump.sh + paste as session startup**: manual, fragile, Adam-dependent.
  Replaced by Cowork's native memory system.
- **pr_dump.sh as the only way to get a diff to a reviewer**: necessary
  only because Clead had no GitHub access. Replaced by direct GitHub
  fetch via Cowork connector.
- **The `?i=1` re-report convention**: existed only to track clipboard
  relay iterations. Disappears when the relay disappears.
- **The hard stop rule**: existed only to prevent the implementer from
  acting on reviewer comments without Adam as the gate. Disappears when
  Clead communicates directly with Crog via GitHub.
- **STOP-and-wait scaffolding**: same root cause as hard stop rule.
- **Adam relaying fix prompts**: pure relay work, zero value added.
- **Adam posting Clead's verdict as PR comment**: pure relay work.
- **Adam updating CHANGELOG after merge**: automatable, no judgment
  required.
- **Adam running dump.sh and pasting**: replaced by Cowork memory.
- **Ten-page TEAM_STRUCTURE.md**: most of it documents workarounds.
  Replace with one page of actual principles.
- **Multiple NEXT_SESSION files**: symptom of a memory problem that
  Cowork's memory system solves structurally.

---

## 4. The New Team

Three roles. Minimum viable. No role exists unless it is doing something
a different role cannot do.

### Adam — Product Owner
- Decides what gets built and in what order
- Approves the merge
- Overrides any decision if needed
- Steps in for genuinely novel architectural decisions where domain
  judgment is required
- **That is all.** Every other touchpoint is eliminated.

### Clead — Tech Owner / Reviewer (runs in Cowork)
- Maintains persistent memory of the project via Cowork's memory system:
  project description, review standard, current PBI, key decisions
- Produces or validates the spec before implementation starts
- Reviews every PR: fetches diff directly via GitHub connector,
  applies the Clead Review Standard, posts verdict as PR comment directly
- Triggers Crog via Routines once secrets support is available;
  until then, Adam fires Crog on Clead's instruction (one paste,
  not a relay loop)
- Posts fix prompts directly to the PR as a comment for Crog to pick up
- Runs in Cowork chat for thinking/planning sessions with Adam
- Runs autonomously for review loops

### Crog — Implementer (runs in Claude Code / Routines)
- Receives task via PR comment from Clead, or via Adam paste (transitional)
- Implements TDD-first: tests red, implementation green, refactor
- Opens PR, posts verdict-ready comment for Clead to pick up
- Picks up Clead's fix prompts from PR comments, implements, pushes
- Updates CHANGELOG as part of the merge step
- Never merges — waits for Adam

---

## 5. The New Workflow

### Session startup (Cowork replaces dump.sh)
Clead's memory is persisted in Cowork's file-based memory system
(`CLAUDE.md` hot cache + `memory/` directory). At session start,
Cowork loads this automatically. No dump file. No paste. No ritual.

The memory files contain:
- Project description and goals
- Current PBI and backlog state
- Clead Review Standard (verbatim)
- Key architectural decisions
- Role definitions and rules (one page, not ten)

### PBI execution loop

```
Adam picks next PBI
        ↓
Adam describes it to Clead in Cowork chat (one sentence to one paragraph)
        ↓
Clead elaborates into a task spec if needed, confirms with Adam
        ↓
Clead fires Crog (via Routines curl, or Adam pastes — transitional)
        ↓
Crog implements TDD-first, opens PR, posts pr_done comment on PR
        ↓
Clead detects new PR (polls or is triggered), fetches diff via
GitHub connector, runs Clead Review Standard
        ↓
Clead posts verdict as PR comment
        ↓
    [If changes needed]
    Clead posts fix prompt as PR comment
    Crog picks it up, implements, pushes
    Clead re-reviews
    Loop until approved
        ↓
    [If approved]
    Clead posts approval comment
    Adam reviews approval comment, merges
        ↓
Crog updates CHANGELOG as post-merge step (triggered by merge event
or by Adam's single instruction)
        ↓
Next PBI
```

Adam's touchpoints in this loop: **two**. Describe the PBI. Approve
the merge. Everything else is machine-driven.

### Transitional state (until Routines secrets support lands)
Until Anthropic adds secrets injection to Routines, Adam cannot be
fully removed from the Clead→Crog trigger. The transitional flow:
Clead produces a curl command, Adam substitutes the token from his
password manager and fires it. This is documented in `docs/ROUTINES.md`
and is already in use. It is one paste, not a relay loop — significantly
better than the current state even if not yet fully autonomous.

---

## 6. What Replaces the Current Documentation

Current template has: BOOTSTRAP.md, CLAUDE.md, CROG_ONBOARDING.md,
TEAM_STRUCTURE.md, DEV_INFRASTRUCTURE.md, PRODUCT_BACKLOG.md,
CHANGELOG.md, dump.sh, pr_dump.sh, dump-wrapper.sh, multiple
NEXT_SESSION files, and a growing docs/ directory.

New template needs:

| File | Purpose | Replaces |
|---|---|---|
| `CLAUDE.md` | Crog's auto-load entry point — one paragraph | Current CLAUDE.md |
| `docs/CROG_ONBOARDING.md` | Crog's rules: TDD, branch, PR flow, review standard. One page. | Current CROG_ONBOARDING.md (minus workaround scaffolding) |
| `docs/SPEC.md` | Technical product specification | Current TPS (unchanged in structure) |
| `docs/BACKLOG.md` | PBI list | Current PRODUCT_BACKLOG.md (unchanged) |
| `CHANGELOG.md` | Change history | Current CHANGELOG.md (unchanged) |
| Cowork memory files | Clead's persistent context | dump.sh + NEXT_SESSION files + most of TEAM_STRUCTURE.md |

Files that do not exist in the new template:
- `tools/dump.sh` — replaced by Cowork memory
- `tools/dump-wrapper.sh` — same
- `tools/pr_dump.sh` — replaced by Clead's direct GitHub fetch
- `docs/TEAM_STRUCTURE.md` — replaced by one-page rules in CROG_ONBOARDING.md
  and Cowork memory
- `docs/CLEAD_ONBOARDING.md` — replaced by Cowork memory
- `docs/NEXT_SESSION.md` — replaced by Cowork memory

`tools/pr_dump.sh` may be retained as a fallback for sessions where
Clead runs in Chat rather than Cowork. It should not be load-bearing
infrastructure.

---

## 7. The Reviewer Independence Problem

This must be solved structurally, not by convention.

**The risk:** if Clead reviews a PR in the same Cowork session where it
also helped design the feature, it is not an independent reviewer. It
knows too much. The review degrades to rubber-stamping.

**The solution:** Clead's review must be triggered as a fresh context
load — not a continuation of the implementation conversation. In
practice: Clead's review is triggered by the PR opening event. At that
point Clead loads only the diff, the TPS, and the Clead Review Standard.
It does not load the conversation history from the implementation session.

This is the one architectural constraint that must be enforced in the
new design. Everything else is tooling. This is a principle.

---

## 8. What This Is Not

This is not a proposal to remove the review gate. Every PR still goes
through Clead. The review standard is unchanged.

This is not a proposal to automate merge authority. Adam merges.
Always.

This is not urgent. The current workflow functions on all active projects.
This rebuild applies to new projects and to the template. Existing
projects (titan-comptracker, python-blackjack, fomo-f) are not forced
to migrate — they inherit the new workflow only if Adam chooses to
update them.

---

## 9. Decisions Required

1. **Confirm the rebuild decision**: are we rebuilding the template
   from scratch, or iterating on the current one?

2. **Cowork as the primary Clead surface**: confirmed in the session
   that produced this document. Any objections before we commit?

3. **Reviewer independence implementation**: how exactly is the review
   context isolated? Options:
   - Separate Cowork session for review (cleanest, most isolated)
   - Same session but explicit context reset (simpler, less certain)
   - Dedicated Clead Routine for review only (most autonomous)

4. **Transitional plan for active projects**: do titan-comptracker,
   python-blackjack, and fomo-f migrate to the new workflow, or
   continue on the current one until natural end-of-life?

5. **Pilot scope**: which project gets the new template first?
   Recommend a new project rather than migrating an active one.

---

## 10. The One-Sentence Summary

We built a complex system to compensate for constraints that no longer
exist; the rebuild discards the compensation and keeps the principles,
producing a workflow where Adam picks the next PBI, describes it in one
sentence, and approves the merge — everything else is machine-driven.
