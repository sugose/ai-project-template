# Session Structure: Controlling the Uncontrollable
## A 45-minute conference talk — narrative arc with speaker notes

---

## BEFORE YOU GO ON STAGE

This talk has three layers. The audience will only notice two of them.

The first two thirds are pure engineering. You are one engineer talking to other engineers about real problems and real solutions. This is where your credibility is built. Do not rush it. Do not signal that something bigger is coming. Just be an engineer.

The last third begins with one sentence that connects the technical work to something else. That sentence is: *"Teams that can focus on the right things are the teams that are likely to perform."* That is the only bridge. After that sentence, you do not explain. You do not elaborate. You let the tear-jerker do its work.

The audience will not see the transition coming. That is the point.

---

## SEGMENT 1: THE OPENING (5 minutes)

**What this segment does:** Disorient, then ground. The audience should not know what they're in for until you introduce yourself.

---

**Slide 0 — The image (fullscreen, no text)**

Show the image. Say nothing. Let it sit for 5-10 seconds.

Then, completely deadpan:

*"This picture is designed to give the viewer the simulated experience of having a stroke."*

*(pause)*

*"I'd like to think what follows will be slightly less disorienting than that — but for liabilities sake I can make no promises."*

*(beat)*

*"My name is Adam Smolowicz. I run a company called Divbyzero Consulting."*

*(pause — let them work out the name)*

*"Division by zero causes an exception. We are exceptional."*

*(gesture toward your photo on screen)*

*"And as you can see here — this is a picture taken before the introduction of AI. My hair is now all grey."*

*(let that land. do not explain further.)*

---

**Slide 1 — Title**

Move to the title slide. Continue without pause.

*"I've spent most of my career in agile — coach, scrum master, team lead, product owner, product manager. Across industries, across teams. I started programming back in the days when each line of code was pre-pended by a number, preferably divisible by ten."*

*(pause for the smiles)*

*"I have a leading star that guides how I work. It goes like this: my leading star doesn't tell me what we should do next to maximise customer value. It tells me what I can do today so the team I'm working with actually functions well."*

*(brief pause — no elaboration, no reaction sought. plant the seed and leave it.)*

*"I'll come back to that. Right now I want to talk about something that will feel much more familiar to most of you in this room."*

*(beat)*

*"A GitHub Copilot integration that didn't work the way it was supposed to."*

*(move to slide 2)*

---

**Slide 2 — The team**
Four roles. Simple diagram or list.
- Product owner (human, authority)
- Tech lead (AI — architecture, review)
- Developer (AI — implementation)
- Copilot (automated reviewer)

**Speaker note:** *"The human is the only one who merges. The AIs do everything else. That sentence will make more sense by the end of this talk."*

---

## SEGMENT 2: THE FIRST PROBLEM — RE-REVIEW (10 minutes)

**What this segment does:** Establish technical credibility. This is a real problem that many people in the room have hit. When you describe it accurately, they will nod.

---

**Slide 3 — "The first review works"**
Simple flow: PR open → Copilot reviews → findings appear.

**Speaker note:** *"The first time Copilot reviews a PR, it is useful. It catches real things. Style violations, test gaps, cross-file inconsistencies visible in the diff. On a multi-iteration workflow — where a PR might go through four fix rounds before merge — the first review is valuable."*

---

**Slide 4 — "The second review is the problem"**
Same flow, but the second and third review steps are blank or marked with a question mark.

**Speaker note:** *"With 'Review new pushes' enabled in the ruleset, Copilot is supposed to re-review when new commits are pushed. In practice, it does not. Community reports suggest it fails to re-review on roughly one in three pushes. In our workflow, it was consistent: Copilot would review at PR open and then produce nothing."*

*"We tried everything documented. The REST API. GraphQL. The CLI. All of them work for the initial request. All of them fail after the first review. 422. Silent no-ops. Nothing."*

*"The only reliable path is clicking 'Re-request review' in the GitHub UI. That is not acceptable in an automated workflow."*

---

**Slide 5 — "What we discovered"**
Two-column: what we tried (left), what happened (right). Or a simple narrative paragraph.

**Speaker note:** *"We investigated the failure mode carefully. What we observed — across multiple PRs, consistently — was this: a documentation-only push between two source code pushes triggered re-review on the subsequent source push, even when a direct source-to-source push did not."*

*"Our hypothesis is that Copilot tracks an internal baseline of its last reviewed commit, and uses the diff from that baseline to decide whether to run. A documentation push shifts the baseline without triggering a review. The next source push lands as genuinely new content. Copilot fires."*

*"We posted this to the GitHub community forum. We have two data points. It is not yet confirmed."*

---

## SEGMENT 3: THE SOLUTION — LABEL-BASED REVIEW (10 minutes)

**What this segment does:** Show the design decision. This is where the engineering thinking lives.

---

**Slide 6 — "Event-driven vs interrupt-driven"**
Two columns or two states:
- Event-driven: runs on every push
- Interrupt-driven: runs when a human decides it adds value

**Speaker note:** *"The re-review investigation produced one insight that changed how we think about the whole problem: Copilot was being used as an event-driven tool when it should be used as an interrupt-driven tool."*

*"Event-driven: run on every push. Interrupt-driven: run when a human decides the value exists."*

*"That distinction sounds simple. It has significant consequences."*

---

**Slide 7 — The workflow**
Show the YAML. Not all of it — the key parts: `labeled` trigger, `ai-review` guard, the two steps (request review, remove label), the `if: always()` on the cleanup.

**Speaker note:** Walk through it briefly. *"Adding the `ai-review` label triggers exactly one Copilot review. The label is automatically removed — even if the run fails. A concurrency block prevents parallel runs. 'Review new pushes' is disabled."*

---

**Slide 8 — The label lifecycle rules**
Five or six invariants, cleanly listed.

**Speaker note:** *"These are not suggestions. They are invariants. The system fails in specific ways if you violate them — usually by burning tokens on a review that tells you nothing new, or by getting into a state where you can't tell whether a review is pending or complete."*

*"The important one is this: re-review is permitted only after changes affecting program behaviour. Logic changes. Interface changes. Data flow changes. Formatting does not justify re-review. A comment change does not justify re-review. Spending credits on a review of a change that couldn't possibly affect correctness is waste — and the rule makes that waste explicit."*

---

**Slide 9 — "What this actually saved"**
Token reduction: ~70-80% vs "Review new pushes" model. One or two deliberate reviews per PR instead of five automatic ones.

**Speaker note:** *"The credit reduction was meaningful. But the more important change was signal quality. When you run Copilot five times on a PR that iterates five times, at least three of those runs produce noise — findings about code that has already been fixed, or findings that don't change the outcome. When you run it twice, deliberately, at moments where the review matters, the findings are worth reading."*

---

## SEGMENT 4: THE SECOND PROBLEM — WHAT COPILOT ACTUALLY IS (8 minutes)

**What this segment does:** Reframe the tool. This segment plants the seed that will germinate in the last third — but do not signal that. Stay in engineering mode.

---

**Slide 10 — "Copilot is not a reviewer"**
Single large statement on a dark slide.

**Speaker note:** *"Running Copilot as a label-triggered tool forced a more honest accounting of what it actually does. And what it does is not code review. Not in the way a human reviewer would review code."*

*"Copilot reviews the PR diff. It does not review the codebase. It cannot detect inconsistencies in code that is not part of the diff — even if the change logically affects it."*

---

**Slide 11 — The interface bug**
Brief description of the incident: strategy interface changed in one file. Dependent file not in diff. Copilot found nothing wrong. Tech lead cross-file check caught the bug.

**Speaker note:** *"On one PR, a strategy interface change. Copilot reviewed the modified file correctly. Found nothing wrong. Did not flag that a dependent file — not in the diff — still called the old interface with the old signature."*

*"The bug would have shipped. The tech lead's cross-file consistency check caught it."*

*"Copilot's review was accurate within its scope. Its scope was too narrow."*

---

**Slide 12 — "Heuristic analyser, not reviewer"**
Three-layer model:
- Layer 1: Copilot — fast, shallow, diff-bounded, advisory
- Layer 2: Tech lead — deep, contextual, cross-file
- Layer 3: Process — controls when each layer runs

**Speaker note:** *"The correct mental model is: Copilot is a heuristic analyser. It provides fast, targeted, advisory input on visible diff content. Its output is never authoritative. It catches what it can see. For what it cannot see — you need someone with full context."*

*"That someone can be a human tech lead. Or, in our case, an AI tech lead with access to the full codebase. Either way, the role exists. It is not covered by Copilot."*

---

## SEGMENT 5: THE THIRD PROBLEM — MULTI-AGENT COORDINATION (7 minutes)

**What this segment does:** The most honest part of the talk. This is where you describe the failure modes that nobody else will admit to.

---

**Slide 13 — "The hard stop rule"**
Single rule, stated plainly: after posting the PR dump, the developer stops completely.

**Speaker note:** *"The most important rule in our process is one I did not know I needed until the process broke without it."*

*"After the developer agent opens a PR and posts a structured dump of the PR state as a comment, it stops. Completely. It does not read the reviewer's comments. It does not push fixes. It does not act on anything it observes. It waits."*

*"Every iteration is gated by the tech lead. Nothing autonomous happens between the dump and the verdict."*

---

**Slide 14 — "Why the rule exists"**
Brief description of the PR #58 incident.

**Speaker note:** *"This rule exists because of a specific failure. An early PR — before the rule was formalised. The developer agent observed Copilot's comments, interpreted them, and pushed what it thought were fixes. The fixes were incorrect. They introduced a subtle spec change that was not requested. The spec drift was invisible until three PRs later."*

*"Unwinding it cost more time than the original fix would have taken."*

*"An AI agent that acts on its own interpretation of reviewer comments — without human oversight — introduces spec drift at a rate that compounds silently. The constraint feels slow. The alternative is slower."*

---

**Slide 15 — "The process is the product"**
Simple statement: when the process breaks, it gets fixed in a PR.

**Speaker note:** *"Every failure produced a process change. Every process change was documented. The PR history is the failure history. That is not embarrassing — it is the mechanism by which the process improves."*

*"The teams that will use AI well are the teams that treat AI failures as process failures and fix them accordingly."*

---

## SEGMENT 6: THE TURN (4 minutes)

**What this segment does:** The hinge. Still fully technical. Already carrying the human truth. The audience will feel it before they understand it.

---

**Slide 16 — Ambiguity**

*"What we built wasn't a smarter system."*

*(pause)*

*"It was a system where you don't have to guess. You know when it runs. You know what it does. You know when it's done. Nothing hidden. Nothing implicit."*

*(pause)*

*"What we had before wasn't a broken system."*

*(longer pause)*

*"It was an ambiguous one."*

---

**Slide 17 — The silence**

*"You didn't know what silence meant."*

*(pause)*

*"And once you don't know what silence means —"*

*(pause)*

*"you stop thinking about the work."*

*"You start thinking about the system."*

*(do not elaborate. move to next slide.)*

---

**Slide 18 — What it does to you (negative words)**

Say nothing. Let the words sit.

After 5 seconds:

*"Who performs well feeling like this?"*

*(pause. do not answer. move to next slide.)*

---

**Slide 19 — The rename + positive words**

The title has changed. No announcement. Just let them see it.

*"The Unsettling Silence."*

*(pause)*

*"...what it does to you."*

*(pause — let them look at the positive words)*

*"When a team feels like this consistently —"*

*(pause)*

*"we call that team health."*

*(beat)*

**Slide 20 — Transition line (dark, single line)**

*"Teams that can focus on the right things are the teams that are likely to perform."*

Say it once. Pause. Move directly to the tear-jerker.

---

## SEGMENT 7: THE CLOSING (4 minutes)

**What this segment does:** The tear-jerker. Exactly as written. Pacing is everything. Do not rush. Do not explain what you are doing. Trust the audience.

---

**Slide 17 — Dark slide. No text.**

**Speaker note — deliver this exactly:**

*"So... did this actually create happier teams?"*

*(pause)*

*"Yes."*

*(pause)*

*"But not in the way I expected."*

*"We didn't do anything to make people happier."*

*"We didn't run workshops."*

*"We didn't talk about culture."*

*"We didn't try to 'improve team health'."*

*(pause)*

*"We just... removed the things that made work harder than it needed to be."*

*"We removed the uncertainty."*

*"We removed the guessing."*

*"We removed the need to think about the system... instead of the problem."*

*(pause)*

*"And something changed."*

*"People stopped checking things."*

*"They stopped second-guessing."*

*"They stopped compensating."*

*"They just... worked."*

*(long pause)*

*"And that's when it hit me."*

*"My leading star doesn't tell me what we should do next to maximize customer value."*

*(pause)*

*"It tells me what I can do today... so the team I'm working with actually functions well."*

*(pause — let them think "this is where it's going")*

*"Because a healthy team..."*

*(longer pause)*

*"...is far more likely to perform."*

*"And performance..."*

*(slower now)*

*"...is what creates customer value."*

*(Do not explain it. Do not add anything.)*

*(Stand still for three seconds.)*

*"Thank you."*

---

## SLIDE 22 — Final slide (Paco)

Paco's photo fills the left half of the screen. His quote is hidden. It will appear on your next click.

**Speaker note:**

Let the slide appear. Let them look at him.

*"This is Paco. Our CEO."*

*(pause — let them register the cone of shame)*

*"Paco only has one ear. It's so he doesn't have to listen to half the crap you tell him."*

*(click — his quote fades in underneath him)*

*"I told you it wouldn't work." — Paco, CEO*

*(let it sit. a few seconds. let the room read it.)*

*"It wasn't until very recently — long after I put this presentation together — that I realized our beloved CEO was actually right."*

*(three seconds of silence)*

*"Thank you."*

---

Do not elaborate on what he was right about. Do not explain. Do not add anything.

Walk off.

Every person in that room will finish the thought themselves. That is the point.

---

## PACING NOTES

| Segment | Time |
|---|---|
| Opening (stroke image + intro) | 5 min |
| First problem: re-review | 10 min |
| Solution: label-based | 10 min |
| Second problem: what Copilot is | 8 min |
| Third problem: coordination | 4 min |
| The turn (ambiguity → silence → words → rename) | 4 min |
| Transition line | 30 sec |
| Closing (tear-jerker) | 4 min |
| **Total** | **45 min** |

Leave the last 4 minutes of a 45-minute slot for questions — but do not announce this before the tear-jerker. End the talk. Let the silence sit. Then open the floor.

---

## WHAT TO PROTECT AT ALL COSTS

1. The transition sentence is one line. Not two. Not a paragraph.
2. The tear-jerker has pauses. The pauses are not awkward. They are the mechanism.
3. Do not explain what just happened after the closing. Do not say "so what I was trying to show you there was..." Just stop.
4. The article and the talk cover the same material. The article does not hint at the closing. Someone who reads the article before the talk will not see the transition coming. Keep it that way.
