---
name: feedback
description: 'Close the loop after a blitz: look at what my loops found, what was built from it, what merged and what it cost, then ask me what was worth it and what was not, and turn my answers into where the loops look next. Use after a blitz has landed, when I ask how the last round went, what was worth it, which loops are earning their keep, or say a loop is too noisy, too quiet or looking in the wrong place.'
allowed-tools:
  - mcp__tekk__get_workspace_overview
  - mcp__plugin_tekk_tekk__get_workspace_overview
  - mcp__tekk__list_sessions
  - mcp__plugin_tekk_tekk__list_sessions
  - mcp__tekk__fleet_status
  - mcp__plugin_tekk_tekk__fleet_status
  - mcp__tekk__list_specs
  - mcp__plugin_tekk_tekk__list_specs
  - mcp__tekk__get_spec
  - mcp__plugin_tekk_tekk__get_spec
  - mcp__tekk__list_proposals
  - mcp__plugin_tekk_tekk__list_proposals
  - mcp__tekk__list_runs
  - mcp__plugin_tekk_tekk__list_runs
  - mcp__tekk__get_run
  - mcp__plugin_tekk_tekk__get_run
  - mcp__tekk__spend_report
  - mcp__plugin_tekk_tekk__spend_report
  - mcp__tekk__get_steering_brief
  - mcp__plugin_tekk_tekk__get_steering_brief
  - mcp__tekk__list_steered
  - mcp__plugin_tekk_tekk__list_steered
---

Feedback — the last step of the cycle. My loops found things, I swept them, a blitz built them. Now find out what that was worth, ask me what I think, and turn my answers into where the loops look next. Without this step the loops only ever hear my verdict from BEFORE anything was built, when I was guessing.

**The cycle:** `/sweep` decides what the loops found → `/blitz` builds it → `/feedback` judges what came back and re-aims the loops → the loops run → `/sweep` again. Each step offers the next one; none starts it without my yes.

**What feedback is NOT:** a stored note the loops re-read forever. Tekk aims every run fresh from the evidence: what changed, what was accepted, rejected or marked marginal, which areas went unread. A sentence frozen today would be wrong in a month and nothing would correct it. So my verdict reaches the loops only through the things they already read: a steered assignment for a loop's next run, a change to a loop's cadence or autonomy, and how I decide the next findings. Never offer to "save" feedback anywhere else.

## 1. Read the cycle — find the facts yourself

Work out the window: it starts when the earliest session of the last blitz started and runs to now. Use that same start as `since` for `spend_report` and as the cut-off when reading `list_runs`. Never ask me for any of this.

- `list_sessions` (and `fleet_status` for what is still running) — which sessions ran, on which spec, and whether each finished, stalled or was abandoned.
- `get_spec` on each — its `sourceLoop` (the loop the finding came from, or none if I wrote it), its `linkedPrs` (what merged) and its `drift` (whether a merge claimed to finish it, and whether the checklist was done).
- `get_workspace_overview` once — its `loopDecisions` is the kind of no I gave each loop: per loop, over the last 90 days, how many of its findings I decided (`decided`) and how many I rejected as `wrong` (untrue), `wrongFix` (true, but the remedy was wrong) or `tooSmall` (true, not worth a card). A loop with none of those rejections is left out, so a missing loop means none were rejected that way, not that I decided nothing; take its decided count from this cycle's `list_proposals` instead. An entry named `goal` holds findings that came from no loop, so never report it as a loop. These are MY recorded verdicts, so use them rather than asking me to remember; they span 90 days and every project, not just this cycle, so say so when you quote them.
- `list_proposals` with `status: "accepted"` and `"rejected"` — WHICH of this cycle's findings I took and which I turned down, each with `decision`: my own last accept or reject, the kind of no (`wrong`, `wrong-fix`, `not-now`, `already-done`, `too-small`…) and the reason I typed. Quote that reason back rather than asking me for it, and ask about a specific reject only when its `decision.note` is empty. Never read the kind of no off the row's `category`: that is the finding's own topic.
- `list_runs` per loop that appears, with `limit: 100`, and `spend_report` for the same window per loop — how often each ran, how many runs came back empty, and what it cost. `list_runs` returns the newest runs and takes no date filter, so if even its oldest row falls inside the window the count is partial: say "at least N runs", and draw no conclusion about frequency or empty rate from it. `spend_report` covers the whole window, so cost is the complete figure.
- `get_steering_brief` once — what already stands out about the loops, and any assignments still waiting (`waiting`). `list_steered` shows the waiting queue if you need it on its own.

If there was no blitz to look back on, say so in one line and use the last 14 days as the window instead: what each loop found, what I accepted, what merged.

## 2. Show me the cycle, per loop

One short block per loop that produced anything, in order of what it cost:

> **security** — 4 findings, 3 accepted, 2 built and merged, 1 session stalled. 9 runs, 5 empty. ~$3.10. Over 90 days: 2 of 11 decided were rejected as wrong.

Then two or three lines in your own words on what stands out: a loop whose findings all merged, a loop whose work all stalled, a loop that costs a lot and finds little, a loop that found nothing in an area I keep shipping into. Not a status table — I can read a table on the Loops page.

## 3. Grill me in rounds

Ask the whole set of questions that are answerable NOW, in one numbered round, each with your recommended answer. Then stop and wait.

```
❓ **Q1** - **<short title>**: <the question, about this cycle, in my words>

➡️ <your recommended answer, from the facts in step 1>

---
```

What the round is for — ask only the ones the facts give you a reason to:

- **What was worth it.** Which merged fixes mattered: a real problem I did not know about, or something I already suspected? Recommend from what the change touched.
- **What was too small.** Which built items were not worth a session: true, but cosmetic or premature.
- **What was wrong.** Which findings turned out untrue once someone built them, and why.
- **What nobody looked at.** Which part of the product I worried about this cycle that no loop touched.
- **Cadence and cost.** A loop that runs often and comes back empty, or one I would pay more to hear from.

When an answer reshapes the rest, recompute what is askable and ask the next round. Done when nothing is left that I have not answered or waved off. Push once on a vague answer ("the checkout stuff" — the payment webhook, or the form before it?), take what I say, move on.

## 4. Turn the answers into levers

Reason the shape yourself; never ask me for engine vocabulary.

- **Re-aim a loop** → a steered assignment: the area (a capability, named the way I talk about it, with starting paths if you know them), why it matters, the hypotheses the next run should confirm or refute, and the level (`subsystem` for one capability, `repo` for how the whole thing is built). One topic is one assignment. A loop that just looked at an area does not need sending back there, and a hypothesis a run already refuted is not worth sending again without new evidence. Skip a loop whose `waiting` queue already covers it.
- **Too noisy or too expensive** → a cadence or autonomy change, shown as what it is now and what it would become.
- **Not running at all and should** → switching it on, with what that will cost.
- **The verdict on a finding** → what a decision records happens at decision time. A finding already accepted keeps the worth it was given; there is no tool to change it afterwards, so do not pretend to. Say what this cycle taught for the NEXT sweep instead: which loop's findings deserve a closer look, which reject category fits what came back wrong.

Most loops end a feedback round with no change. That is the normal outcome, not a gap to fill.

## 5. Show it, wait, write

ONE block: per loop, each assignment you would write and each setting you would change, before and after. Then **wait for an explicit yes.** Nothing is written until I confirm.

On a yes, hand the block to the tools that own it: `steer_loop` writes a loop's assignments (a switched-off loop is refused; ask me whether to switch it on), and `configure_loop` changes on/off, autonomy and cadence. Both may need my client to approve the call. If I want to go deeper than this cycle's evidence — where the product is going rather than what just happened — offer `/grill-to-steer` instead and hand it what you learned.

Report anything that was refused, and say in one line when each assignment will run: on that loop's next background wake.

## Next — offer the next stage

End with the next stage of the cycle, as a question: if new proposals have arrived since the sweep, "Sweep them now?" (`/sweep`); if accepted specs are still waiting to be built, "Blitz them?" (`/blitz`). If nothing is waiting, say the loops will run on their own schedule and that `/feedback` is the step after the next blitz. Start nothing without my yes.

---

The round structure — every answerable question at once, each with a recommended answer, finding facts rather than asking for them — is adapted from the `grilling` skill in Matt Pocock's `mattpocock/skills` repository, MIT licensed.
