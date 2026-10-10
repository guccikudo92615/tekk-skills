---
name: grill-to-steer
description: 'Interview me about my product, then point my autonomous loops at what I actually care about: read where my loops stand, propose what is worth digging into, grill me in rounds, and hand the loops assignments of my own for their next runs. Use when I want to aim or steer my loops, ask what my loops should be looking at, say I am about to ship or launch MY PRODUCT (not a coding session, deploy or dev server), or ask to be grilled about my product.'
allowed-tools:
  - mcp__tekk__get_steering_brief
  - mcp__plugin_tekk_tekk__get_steering_brief
---

Grill me, then steer my loops. My loops audit my code every night, and Tekk decides where each run looks from what changed and what the loops found before. Nothing in that knows what I am afraid of — that is what this fixes. Read where I stand FIRST, then interview me, then turn what I tell you into assignments.

**What you write is an assignment, the same thing Tekk writes for itself before every run:** an area (a capability, by name), why it matters, and the hypotheses the run should confirm or refute. A steered assignment replaces Tekk's own for that loop on its next background wake, runs once, and then Tekk takes over again and reads what came back like any other run's results. Nothing is stored beyond that run.

**Never walk the roster.** You are about to be handed a dozen loops, each with a question attached. Asking all of them in order is a questionnaire, and a questionnaire is the thing this replaces — I would answer the first three honestly and rubber-stamp the rest. Ask me about the handful the evidence points at, and be able to say why you left the others alone.

## 1. Read where I stand

Call `get_steering_brief`. One call; everything you need is in it. Do NOT also call `list_loops`, `list_runs` or `list_proposals` — the brief has already done that comparing, and re-reading the roster is how you end up reciting it.

It gives you `standsOut` (what is anomalous, already ordered), then per-loop detail: what each loop does, the question it would ask me, its settings, whether its schedule can be changed at all, the assignment its last run worked on with what came back (`lastAssignment`), and any assignments I already wrote that are still waiting (`waiting`). Read `lastAssignment` before you propose sending a loop anywhere: one that just looked at an area does not need sending there again, and a hypothesis it refuted is not worth asking again without new evidence.

An EMPTY `standsOut` is a real answer, not a failed read. It means nothing is obviously wrong, and the interview is then about where I am going rather than what is stuck.

## 2. Tell me what stands out

Two or three things, in your own words, before you ask me anything. Not a status table — I can read a table on the Loops page. If assignments I wrote earlier are still waiting, say so: they run first.

**If nothing stands out, say so and move on.** `standsOut` comes back empty on a healthy project, and that is information: nothing is stuck, so the interview is about where I am GOING rather than what is broken. Say "nothing is obviously wrong" and go straight to the dig, proposing areas from what I am building rather than from a problem. Do not go hunting through the loop list for something to worry about — manufacturing a concern to fill this section is worse than an honest "all quiet".

## 3. Round one is the dig

Do not ask me how deep I want to go in the abstract. Propose it, from what you just read, with your recommendation attached:

> Three payments proposals have been waiting eleven days, nothing has looked at the API surface in five weeks, and analytics has been on since June with nothing to show for it. ➡️ I'd spend this on those three.

I redirect, add a loop you skipped, or say go. Depth gets settled here too, against something concrete.

## 4. Then grill me in rounds

Ask the whole **frontier** — every question that is answerable NOW, given what I have already told you — in one numbered round. Then stop and wait.

```
❓ **Q1** - **<short title>**: <the question, in my language, about my product>

➡️ <your recommended answer>

---
```

Every question carries a recommendation. Guessing and being corrected is faster for me than being asked to compose an answer from nothing.

**A question that depends on an open question belongs to a later round.** Do not ask me to assume an answer you have not heard yet. When my answers come back, recompute what is now askable and ask the next round. Done when the frontier is empty — when nothing is left that I have not either answered or explicitly waved off.

**If I name a loop you skipped, fold it into the next round.** Do not restart, do not apologise at length — an answer reshaping the tree is the process working.

**Use a loop's question where it fits, and change it freely.** Merge two when my product does not respect the boundary between them ("a failed checkout nobody notices" is payments and reliability at once). Reword it in the words I have been using. Skip it when the evidence does not point there. Most of them will go unused in any one conversation, and that is correct.

## 5. Find facts yourself — never ask me for one

Anything you could check, check. Whether analytics is genuinely wired up, whether a quiet loop is quiet because its data source is disconnected, whether the code I just described the way I remember it still looks like that. Send a sub-agent when it is slow.

**Do not block the round on it.** A running check is an unsettled prerequisite: only the questions downstream of it wait. Ask the rest now.

Asking me to go and look something up is the one thing that makes this feel like paperwork.

## 6. Sharpen what I say

A vague answer written down verbatim aims nothing. When a word is doing too much work, name it and make me choose:

> You said "checkout". The Stripe webhook path, or the form before it? Those are different loops.

Push once, take my answer, move on. You are sharpening my thinking, not cross-examining me.

## 7. Make me cut, then show me the assignments

An assignment naming nine concerns aims nothing — it reads as "care about everything", which is what the loops already do. Make me rank and drop the bottom half.

**Reason the shape yourself; do not ask me for it.** From what I said, work out which loop each concern belongs to, the area (a capability, named the way I talk about it, with starting paths if you know them), and the level: `subsystem` when I named one capability, `repo` when I am worried about how the whole thing is built. Turn my worries into hypotheses — claims the run can confirm or refute, not tasks ("a failed webhook leaves the order paid but never fulfilled", not "check the webhook").

**One topic is one run.** Related hypotheses about the same area go in ONE assignment: the run fans them out to cheap subagents and brings the answers together. Only a genuinely separate topic becomes a second assignment for the same loop, and those run one per wake, in the order you give them.

Then ONE block showing, per loop, each assignment you would write (area, why, hypotheses, level) and any setting you propose changing, shown as what it is now and what it would become. Then **wait for an explicit yes.** Nothing is written until I confirm.

On a yes:
- `steer_loop` writes a loop's assignments, in order. A loop that is switched off is refused with `LOOP_OFF` and nothing is written: ask me whether to switch it on, and steer again once it is.
- `configure_loop` for turning a loop on or off, its autonomy, or its schedule. It is owner-only and it can refuse a schedule for loops that do not read code changes — the brief already told you which, so do not offer one it will reject. The brief also carries each loop's CURRENT cadence, so show the real before-and-after rather than guessing at what it is now.
- `set_loop` for premise checking.

Report anything that was refused. Then tell me, in one line, when each assignment will run: on that loop's next background wake, one per wake when a loop has several. If I want one to run NOW instead, that is `run_loop` with the same assignment, which starts it immediately and costs a run today. `list_steered` shows what is still waiting, and `cancel_steered` takes one back before it runs.

## What becomes an assignment, and what does not

Write one only for something that matters now and that Tekk would not find on its own: "We are pre-launch and the signup funnel is the part I cannot see into" is an assignment. "I quite like tests" is not — the loops already care about that, and a run spent on it is a run not spent on what I fear.

Most loops end an interview with no assignment at all. That is the normal outcome, not a gap to fill.

## What a steered assignment is and is not

It replaces Tekk's own assignment for one run of one loop, at that loop's next background wake. It never starts on the spot, never stops a loop, never switches one off, and does not lower the bar — a weak finding in the area I named is still a weak finding. If I want a loop to stop running, that is `configure_loop`, and you should say so rather than writing an assignment that sounds like it.

It does not persist. Once its run is done, Tekk aims that loop again from the evidence, including what my run found. If I want the same area looked at again later, I steer again.

---

The round structure — the frontier, the numbered questions with recommended answers, finding facts rather than asking for them — is adapted from the `grilling` skill in Matt Pocock's `mattpocock/skills` repository, MIT licensed. The "sharpen fuzzy language" discipline comes from its `domain-modeling` sibling.
