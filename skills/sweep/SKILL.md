---
name: sweep
description: 'Work the whole pending-proposal inbox in one pass: re-check every finding against the code as it is now, say which no longer hold and whether that means fixed or merely moved, then decide the batch in one call once the user says go. Use when proposals have accrued, or the user asks what their loops found.'
allowed-tools:
  - mcp__tekk__list_proposals
  - mcp__plugin_tekk_tekk__list_proposals
  - mcp__tekk__get_proposal
  - mcp__plugin_tekk_tekk__get_proposal
  - mcp__tekk__verify_proposal
  - mcp__plugin_tekk_tekk__verify_proposal
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

## 4. Decide — one pass, on my say-so
Give me a single list: accept / revise / reject, one line of reasoning each. Then STOP and wait.
If I say go:
- Use `decide_proposals` ONCE for the whole accept/reject batch. Do not loop over `accept_proposal`. Each accept lands where its placement says; where I agreed with your suggestion instead, put it on that entry as `placement` — `{ "as": "new" }`, or `{ "into": "<spec>", "as": "subtask" | "group" | "amend" | "duplicate" }` (a `group` needs a `title`). An entry the board cannot take that way comes back `placement_refused` with the reason and stays pending; tell me, do not retry it some other way on your own.
- For each revise, call `revise_proposal` with feedback specific enough to act on ("the finding holds but cites src/old/path.ts; it moved to src/new/path.ts" beats "please update"). Revising is capped at three per proposal and costs credits, so make each one count.
Do NOT call `decide_proposals`, `accept_proposal`, `reject_proposal` or `revise_proposal` before I have said go. Verifying is yours; deciding is mine.

## After
A revise runs in the BACKGROUND: the row goes back to pending with new content in a minute or two. Name what you kicked off, do not re-read those rows in this pass, and tell me they will be ready for the next sweep.
Each accepted result says where it went (`placement`). One that became a new spec, a sub-task or a group member has its body written by a queued run, so it is not immediately buildable either; one folded into an existing spec is being amended the same way; a duplicate created nothing new.
Finish by telling me what is now accepted and where each one landed — which specs grew and which are new — what is being revised, what I rejected, and that `/blitz` is what turns the accepted specs into running sessions once their bodies land. If a placement turns out wrong after the fact, `split_out_finding` makes that finding its own spec again (not for one folded into a body).
