---
name: sweep
description: 'Work the whole pending-proposal inbox in one pass: re-check every finding against the code as it is now, say which no longer hold and whether that means fixed or merely moved, then decide the batch in one call once the user says go. Use when proposals have accrued, or the user asks what their loops found.'
---

Sweep — go through my pending proposals, check whether they are still true, and help me clear them in one pass. These have accrued because deciding them one at a time was never worth it; that is the problem to solve, so work the whole inbox, not the first interesting one.

## 1. Read — what is waiting
Call `list_proposals` (pending). Each row carries `loopSkill` — the loop that raised it — and `evidenceCheck`, a summary of the last time its evidence was verified (`null` means never re-checked since it was written). When the finding stated a premise — the claim its citations were gathered to support — the row also carries `premiseCheck`, its last verdict (`null` means no premise was stated).
Group by loop and show me the shape of the queue: which loops are producing, how much, and how old. Severity will not separate them for you — these are mostly the ones nothing marked urgent, which is why they are still here.

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
Also flag any two proposals that are the same finding from different loops — say which you would keep. Do not merge them yourself.

## 4. Decide — one pass, on my say-so
Give me a single list: accept / revise / reject, one line of reasoning each. Then STOP and wait.
If I say go:
- Use `decide_proposals` ONCE for the whole accept/reject batch. Do not loop over `accept_proposal`.
- For each revise, call `revise_proposal` with feedback specific enough to act on ("the finding holds but cites src/old/path.ts; it moved to src/new/path.ts" beats "please update"). Revising is capped at three per proposal and costs credits, so make each one count.
Do NOT call `decide_proposals`, `accept_proposal`, `reject_proposal` or `revise_proposal` before I have said go. Verifying is yours; deciding is mine.

## After
A revise runs in the BACKGROUND: the row goes back to pending with new content in a minute or two. Name what you kicked off, do not re-read those rows in this pass, and tell me they will be ready for the next sweep.
An accepted proposal becomes a spec whose body is written by a queued run, so it is not immediately buildable either.
Finish by telling me what is now accepted, what is being revised, what I rejected, and that `/blitz` is what turns the accepted specs into running sessions once their bodies land.
