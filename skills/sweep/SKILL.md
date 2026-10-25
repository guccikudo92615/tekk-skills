---
name: sweep
description: 'Work the whole pending-proposal inbox in one pass: re-check every finding against the code as it is now, say which no longer hold and whether that means fixed or merely moved, judge what is left on six pass/fail criteria, then decide the batch in one call once the user says go. Use when proposals have accrued, or the user asks what their loops found.'
allowed-tools:
  - mcp__tekk__list_proposals
  - mcp__plugin_tekk_tekk__list_proposals
  - mcp__tekk__get_proposal
  - mcp__plugin_tekk_tekk__get_proposal
  - mcp__tekk__verify_proposal
  - mcp__plugin_tekk_tekk__verify_proposal
  - mcp__tekk__rank_proposals
  - mcp__plugin_tekk_tekk__rank_proposals
---

Sweep — go through my pending proposals, check whether they are still true, and help me clear them in one pass. These have accrued because deciding them one at a time was never worth it; that is the problem to solve, so work the whole inbox, not the first interesting one.

## 1. Read — what is waiting
Call `list_proposals` (pending). Each row carries `loopSkill` — the loop that raised it — and `evidenceCheck`, a summary of the last time its evidence was verified (`null` means never re-checked since it was written). When the finding stated a premise — the claim its citations were gathered to support — the row also carries `premiseCheck`, its last verdict (`null` means no premise was stated).
Some rows also carry `sameShapeAs` — other proposals that make the same claim — and the page carries `themes`, each one claim with the files every proposal in it cites.
Each row also carries `placement`: where accepting it will put it. Tekk places every finding on the board as it arrives, so most of the deciding about WHERE is already done. A `high`-confidence placement is applied as is on accept — into an existing spec as a sub-task (`subtask`), beside a plain spec under a new group (`group`), folded into an idle spec's body (`amend`), or noted on the spec that already says it (`duplicate`). A `low` one is only a possible home: accepting it makes a new spec unless you choose otherwise. `null` means it is still being placed, and a `failed` one could not be.
Group by loop and show me the shape of the queue: which loops are producing, how much, and how old. Severity will not separate them for you — these are mostly the ones nothing marked urgent, which is why they are still here.
Then say in ONE line what will land on its own, for example "7 will land automatically: 3 into the analytics group, 2 beside the billing spec, 2 as duplicates". Do not walk those one by one; your attention goes to the rest — low-confidence, failed and missing placements — and to anything the checks below flag.

## 2. Verify — is it still true?
For each pending proposal, call `verify_proposal`. It re-runs the deterministic evidence checker — and, when the finding stated one, the premise check — against the project's CURRENT checkout and records the result, so this is a fact-gathering step: do it for all of them without asking.
This matters because the original check ran when the proposal was written. A finding from three weeks ago is still carrying a three-week-old verdict, and the code has moved since.
A proposal with no repository to check against comes back with nothing proven either way — report that honestly rather than treating it as a pass or a failure.

## 3. Triage — fixed, or just moved?
Evidence that no longer resolves means one of two OPPOSITE things, and the whole value of this phase is telling them apart:
- The finding was FIXED — someone already did it. That is a reject, and a useful one.
- The code MERELY MOVED — a refactor, a rename, a file split. The finding may still be entirely true and just cites the wrong lines. That is a revise, not a reject.
Read enough of the current code to say which, and say so per proposal with the failing citations. Never auto-reject on a failed check: a refactor is not a fix, and discarding a real finding because someone moved a file is the expensive mistake here.
A premise that now comes back `failed` means the code changed under the finding: it was fixed, or the claim it rests on is no longer true. Treat it like a failed citation — read the current code and say which. A `held` premise only means nothing was found against it, never that it is proven.
Then work the `themes` from step 1: each is one finding raised more than once, often by different loops about different files under different titles. Present it as ONE finding with several sites, not as separate cards — the claim once, then each proposal with the files it cites — and say which proposal you would keep as the carrier and whether the others' sites belong folded into it. A theme is a recorded match, not a verdict: read the members, and if one does not actually make the same claim, say so and treat it on its own. Two proposals you judge to be the same finding without a recorded match are still worth flagging; say that the match is yours rather than recorded. Do not merge or reject them yourself.
For a low-confidence, failed or missing placement, say where you think it belongs — a spec on the board, beside one, or new work — and why, in one line. That is a suggestion for me, not a decision. A confident placement that looks wrong to you is worth the same line; otherwise leave it alone.

Some rows carry `optionGroup`: alternative ways to do ONE move, filed by the same run. They are one choice, not separate cards. Present the group once, the recommended option first, say what each trades against the others, and recommend exactly one to accept, or recommend rejecting the whole group when the problem no longer holds or none of the ways is right. Accepting one closes the rest as not chosen, which never counts against the loop, so never accept two, and when you accept one do not reject the others yourself. Rejecting every member is one rejection of the upgrade, not one per option. Give a group's members adjacent ranks. A direction filed beside the options is its own decision and not part of the group.

## 4. Judge — six criteria, pass or fail
Verifying tells you whether the citations still resolve. It does not tell you whether the finding is worth building, and a sweep that stops there accepts too much. So judge every proposal still standing against the code, on six criteria, each a plain pass or fail:

1. **Real problem** — the stated mechanism holds in the code as it is now, and the consequence follows from it.
2. **Fix works** — applying the plan as written removes the stated problem.
3. **Focused** — one problem, one fix, nothing unrelated attached.
4. **Measured before built** — no new tables, tools, harnesses or abstractions unless something measured shows the need.
5. **Not already there** — no step redoes or parallels something the code or its tests already have.
6. **Leads with best** — the headline is the most valuable item in the proposal.

Work each proposal in this order — critique first, verdicts after:
1. **Read the code yourself.** Read the proposal's full body with `get_proposal`, then the code it names, and check every claim the plan rests on. A quote or a count formatted as evidence is not proof.
2. **Look for what already exists** before judging not already there: search for helpers, tests and patterns that already do the job.
3. **Write the critique.** Two to five sentences: what you checked, what holds, what does not, with `file:line`. Write it before you decide a single pass or fail.
4. **Then the six pass/fails,** each one following from the critique. A verdict the critique does not support is a verdict to change, not a critique to rewrite. If real problem fails, still judge the other five on what the plan would do.

How to judge, so it does not go soft:
- Judge each criterion on its own before you form any overall view of the proposal.
- Pass or fail, nothing between. No "pass, but", no partial credit. If you cannot establish a criterion from the code, it fails, and its reason says what you could not establish.
- A pass on real problem or fix works names where you checked (`file:line`). A pass you cannot point at is a fail.
- Every fail carries a one-line reason naming the line of code or the step of the plan it is about. "Could be more focused" is not a reason; "bundles a retry guard with the logging fix" is.
- The loop's own severity, importance score and placement confidence are not evidence for any criterion. Do not let them lift a pass.
- If you can start subagents, give the judging to fresh ones, a few proposals each: hand each the proposals' bodies and the repository, not the loops' scores, not the placements and not your triage notes. A judge that has read the author's score anchors on it.

Then give each proposal ONE verdict, which follows from the criteria and not from how much you like it:
- **act on it** — all six pass. The only verdict that recommends accept as written.
- **act on part** — the problem is real, but the plan needs changing first: it fails focused, measured before built or leads with best, one of its steps is already there, or its fix fails in a way a revision can correct. Recommend revise, and write the revision from the failed criteria's reasons: what to drop, what to lead with, what to correct in the fix.
- **not worth it** — the problem is real, and what is left once the unmeasured or already-present steps are removed is too small or not wanted now.
- **wrong** — real problem fails, or the fix does not work and no revision would rescue it.

A proposal whose code merely moved (step 3) is judged against where the code is now, but it still cites the old lines: even when all six pass, recommend revise to correct the citations, never accept as is.

Expect most proposals to land at act on part: loops bundle. If you find yourself recommending accept on more than half the batch, go back over the passes on focused and measured before built before you show me anything.

Show the result per proposal: the critique, then the six criteria as pass/fail, the verdict, and the one-line reason for each fail. Judging records nothing on the board and decides nothing, so do it for the whole batch without asking.

## 5. Rank
RANK the batch. A finding's own importance score was written about that finding alone, before it had any neighbour to compare against, so it barely separates anything: call `rank_proposals` ONCE per project, covering EVERY pending proposal in it, with a distinct `rank` for each (1 = do first) and a one-line `note` saying why it sits there relative to the others. A partial ranking is refused, and that is deliberate — a rank only means something against the whole batch. Rank by what is most worth doing now, using what you just learned: act on it above act on part above not worth it above wrong, then by value within each verdict. An option group is ranked as one unit, placed by the verdict of the option you recommend (or as wrong when you recommend rejecting the whole group), and its members take consecutive ranks; a finding whose claim no longer holds goes to the bottom, and a production-impact figure the finding did not measure (its `impactMeasurement` is empty, or only says it is latent) should be treated as an assumption, not a fact. Like verifying, ranking records an order and decides nothing, so do it without asking.

## 6. Decide — one pass, on my say-so
Give me a single list, in rank order: accept / revise / reject, with the verdict and one line of reasoning each. The verdict sets the recommendation:
- act on it → accept, unless its citations moved (step 3), which makes it a revise.
- act on part → revise, with the failed criteria's reasons as the feedback.
- not worth it → reject as `too-small`, or `not-now` when it is worth doing later.
- wrong → reject as `wrong` when the problem does not hold, or `wrong-fix` when the problem is real and the plan is what fails.
- A failed not already there that covers the whole proposal → reject as `already-done`.
Never recommend accepting a proposal with a failed criterion. If I want it anyway, that is my call to make in my reply, not yours to make in the list. Then STOP and wait.
If I say go:
- Use `decide_proposals` ONCE for the whole accept/reject batch. Do not loop over `accept_proposal`. On a reject, carry my reason and its `category` (`wrong` is the only one that counts against the loop), with the failed criteria's reasons as the reason unless I gave my own. On an accept, set `acceptCategory` only when I have said what it was worth (`caught-something`, `already-suspected`, `marginal`); do not ask me for it on every row — I usually cannot know until it is built, and that question belongs to `/feedback`. When I pick an option from a group, put only that one in it, as an accept; the others close on their own. When I decline a whole group (the problem is already fixed, or no way is right), put every member in it as a reject. Each accept lands where its placement says; where I agreed with your suggestion instead, put it on that entry as `placement` — `{ "as": "new" }`, or `{ "into": "<spec>", "as": "subtask" | "group" | "amend" | "duplicate" }` (a `group` needs a `title`). An entry the board cannot take that way comes back `placement_refused` with the reason and stays pending; tell me, do not retry it some other way on your own.
- For each revise, call `revise_proposal` with feedback specific enough to act on ("the finding holds but cites src/old/path.ts; it moved to src/new/path.ts" beats "please update"). Name each failed criterion and what would make it pass ("drop step 3, nothing measured shows the need; lead with the adapter fix"). Revising is capped at three per proposal and costs credits, so make each one count.
Do NOT call `decide_proposals`, `accept_proposal`, `reject_proposal` or `revise_proposal` before I have said go. Verifying is yours; deciding is mine.

## After
A revise runs in the BACKGROUND: the row goes back to pending with new content in a minute or two. Name what you kicked off, do not re-read those rows in this pass, and tell me they will be ready for the next sweep.
Each accepted result says where it went (`placement`). One that became a new spec, a sub-task or a group member has its body written by a queued run, so it is not immediately buildable either; one folded into an existing spec is being amended the same way; a duplicate created nothing new.
Finish by telling me what is now accepted and where each one landed — which specs grew and which are new — what is being revised, and what I rejected. If a placement turns out wrong after the fact, `split_out_finding` makes that finding its own spec again (not for one folded into a body).

## Next — offer the blitz
End with one question: "Blitz the accepted specs now?" `/blitz` is what turns them into running sessions, once their bodies land. If I say yes, run it; if nothing was accepted, say so and skip the question. Never start it without my yes.
