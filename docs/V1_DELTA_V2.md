# v1 → v2 Delta: What Changed and Why

**Date:** 2026-06-27
**Author:** Clead (Claude, acting as Tech Owner)
**Context:** Written at end of the session that produced ai-project-template-v2.
**Reference:** sugose/ai-project-template-v2

---

## The core reason for v2

Almost all of v1's complexity existed for one reason: Clead and Crog
could not talk to each other, and Clead had no GitHub access. Adam's
clipboard was the only channel between them.

That constraint no longer exists. v2 discards the scaffolding that
compensated for it and keeps the principles that were genuinely good.

---

## Where Adam actively participates (v2)

These touchpoints cannot be eliminated — they require genuine human
judgment.

| Step | What Adam does | Why it cannot be a machine |
|---|---|---|
| Pick the next PBI | Decide what gets built next | Product judgment. Requires intent and domain knowledge. |
| Describe the WHAT | Tell Clead what he wants in plain language | Only Adam knows what he wants. |
| Approve the spec | Confirm Clead's spec captures his intent | Intent verification. Clead cannot confirm its own spec against Adam's unstated goals. |
| Approve the merge | Click merge in GitHub | The one hard human control. Requires judgment that the result serves the product goal. |
| Handle escalations | Decide when the review loop hits the 3-cycle cap | Requires judgment beyond what the review standard can resolve automatically. |

---

## Where Adam no longer participates (previously did in v1)

| Step | What Adam used to do in v1 | What happens in v2 |
|---|---|---|
| Paste Crog task prompt | Copied Clead's prompt and pasted it into Claude Code | Clead fires Crog via curl / Routine directly |
| Drop PR URL into Clead's chat | Copied the PR URL from GitHub and pasted it to Clead | GitHub Actions posts review-ready marker; Clead picks it up |
| Paste Clead's fix prompt into Crog | Relayed Clead's fix instructions to Crog | Clead posts fix prompts directly to the PR; Crog picks them up |
| Post Clead's verdict as PR comment | Copied Clead's verdict and posted it to GitHub | Clead posts directly to the PR |
| Run dump.sh and paste output | Ran the script and pasted the dump into a new Chat session | Cowork loads memory files from the repo automatically |
| Update CHANGELOG after merge | Edited CHANGELOG.md manually | Crog does this as a post-merge step triggered by the changelog workflow |
| Orient a new Clead session | Pasted the full repo dump as the first message | Cowork-Clead cold-starts from memory files — no paste, no ritual |

---

## Transitional state (not yet fully automated)

Two items are still partially manual today, pending confirmation of
two unproven pieces:

| Step | Today | Target | Blocked on |
|---|---|---|---|
| Fire Crog | Adam substitutes token in Clead's curl and fires it — one paste | Fully automated | Routines secret injection from Anthropic |
| Trigger Clead review | Adam pastes PR URL to Clead in Cowork — one paste | Fully automated | GitHub connector write access confirmed |

These are one paste each — significantly better than v1's relay loop,
but not yet fully autonomous.

---

## What was discarded and why

| Discarded | Why it existed | Why it's gone |
|---|---|---|
| `dump.sh` + paste startup | No persistent memory | Cowork loads repo-committed memory files automatically |
| `pr_dump.sh` as primary diff path | Clead had no GitHub access | Clead fetches diffs directly via GitHub connector |
| `?i=1` re-report convention | Tracked clipboard relay iterations | Gone with the relay |
| Hard-stop rule | Gated implementer on Adam | Clead communicates directly with Crog via PR comments |
| STOP-and-wait scaffolding | Same root cause as hard-stop | Same |
| Adam relaying fix prompts / verdicts | Pure relay, zero added value | Direct PR comments |
| Adam updating CHANGELOG | Automatable, no judgment required | Crog post-merge step |
| Ten-page TEAM_STRUCTURE.md | Documented workarounds | One page of principles in CROG_ONBOARDING.md |
| Multiple NEXT_SESSION files | Memory problem | Repo-committed memory files |

---

## What was kept and why

These are genuine process controls, not workarounds. They belong in
any rebuild.

| Kept | Why |
|---|---|
| Role separation | One agent implements, a structurally separate agent reviews. Different inputs, not just a different prompt. |
| TDD discipline | Red, green, refactor. Catches real bugs. Non-negotiable. |
| Branch protection + CI as the hard gate | Nothing merges without lint, format, tests, and coverage passing. Enforced at GitHub level. |
| Adam as sole merge authority | The one genuine human control. A judgment point, not friction. |
| The Clead Review Standard | Threat model, spec compliance, error handling pass, test quality check, what was not checked. The right review framework. |
| GitHub as single source of truth | If it is not in the repo, it does not exist. This rule now governs memory too. |
| One task at a time | Complete, review, merge, then next. No accumulating unreviewed changes. |
| Spec precedes implementation | A validated spec exists before Crog writes a line. |

---

## What was added in v2

| Added | Why |
|---|---|
| Repo-committed memory files (`memory/`) | Persistent Clead context that survives session boundaries and works for any clone |
| GitHub Actions as event bus | Drives the review loop without requiring a polling session |
| Dedicated review Routine with scoped inputs | Structural reviewer independence — inputs are {diff, SPEC.md, Review Standard} only |
| Circuit breaker — 3-cycle loop cap | Safety control that re-adds Adam at the right point (judgment call) without re-adding him at the wrong ones (relay) |
| Migration branch in bootstrap wizard | Any team migrating from a previous workflow gets the right questions asked at setup |
| `docs/decisions/` ADR log | Documents why controls exist so they cannot be silently removed or re-accumulated |
| Dual-audience `CLAUDE.md` | Serves both Clead (router/index) and Crog (entry point stub) without conflating them |

---

## The collaboration model (new in v2)

The WHAT/HOW split is made explicit and precise in v2.

**Adam owns the WHAT. Always.** Everything Adam brings — at whatever
level of abstraction or technical depth — is the WHAT.

**Clead owns the HOW.** Everything from Adam's expression to working
code is Clead's responsibility, regardless of gap width.

**The gap is multidimensional** — three independent axes:
- Domain knowledge
- Process knowledge (Adam is strong here)
- Technical knowledge

A technically or process-experienced Adam may narrow the gap
significantly. He may push back on a proposed HOW as over-engineered.
Both are WHAT signals. Clead does not defend HOW choices against them.

**Spec approval is an intent check, not a technical check.** Adam
confirms the spec captures his WHAT. Clead is responsible for
technical correctness. These are different checks and together they
cover the real failure modes.

---

## Touchpoint count — v1 vs v2

| State | Adam touchpoints per PBI |
|---|---|
| v1 | ~8-10 (pick PBI, paste prompt, drop URL, paste fix prompt x N, post verdict, merge, update CHANGELOG, orient new session) |
| v2 today (transitional) | 5 regular + 2 transitional + escalations |
| v2 target | 5 regular + escalations |

The 5 regular touchpoints are irreducible — they all require genuine
human judgment. Everything else has been or will be eliminated.

---

## Reference

- Full ADR: `docs/decisions/0001-rebuild-workflow.md` in this repo
- v2 template: https://github.com/sugose/ai-project-template-v2
- v2 memory files: contain the full session context from 2026-06-27
