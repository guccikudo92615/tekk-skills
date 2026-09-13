---
name: blitz
description: 'Turn what the loops raised and what is already on the board into parallel coding sessions: cluster into the biggest collision-safe batch, open one session per spec, and register each so the fleet stays visible. Use when the user wants to start their planned work, parallelize, or "run through everything".'
---

Blitz — turn what my loops found and what I've planned into parallel work, and start it. I have ALREADY planned this; read it and work with it — do not ask me to re-describe or re-plan anything.

## 1. Sweep — what is actually there
Read BOTH halves of the pipeline before planning anything:
- `list_proposals` (pending) — what the loops have raised and I haven't decided yet. Each row carries `loopSkill`: the loop that found it.
- `list_specs` — the work already accepted or written by hand. Each carries `sourceLoop` (the loop it came from, or null if I wrote it). Consider all my open specs (the whole board).
Open by telling me what each loop has been finding — loop name, how many, one line each. I want to see my loops' week before I see a plan.

## 2. Cluster — collapse, then fan out
Parallelism is not "one session per row". Work out:
- WITHIN a loop: findings pointing at the same code are ONE session, not three colliding ones. Say which you merged and why.
- ACROSS loops: findings touching disjoint areas are the parallel batch — that is where the real speedup is.
- Call `plan_batch` for the deterministic collision-free set. It is the AUTHORITY on what may co-schedule: never override it. Its packages carry `sourceLoop`, `branch`, `repo` and `touches`. For specs with no declared `touches`, judge the file areas yourself from the spec body plus a quick look at the repo.
- Name what must SERIALIZE and why (a dependency, or the same files), so I know the order.
If a spec is big and would fan out better as independent pieces, or two specs should declare distinct file-scopes to run together, POINT THAT OUT and ask before you split or re-tag anything.

## 3. Launch — use the best tier you actually have
Check your own tools and take the FIRST of these you can:
- TIER 1 — you have a tool that OPENS A NEW CODING SESSION from a prompt (and ideally a repo and a branch). Open one per spec in the batch. Use the package's `branch` and `repo`. NEVER give a child a permission mode that blocks waiting for human approval — every session would silently park at an approval screen while I think a fleet is running.
- TIER 2 — you have a tool that QUEUES A TASK for me to start with one click. Queue one per spec. Each prompt must stand alone: the new session starts from that text on a checkout that may not have anything from this one.
- TIER 3 — neither. Hand me a paste-ready prompt per session: the launch line (`claude --worktree='<branch>'` in a new terminal tab, or a new chat on that branch) and then the work prompt.
Say which tier you used. Falling back is fine and silent — do not treat a missing tool as an error.
The work prompt, whichever tier: "Implement Tekk spec TEK-n end to end (use get_spec TEK-n for the detail). Open a PR whose body includes 'Closes TEK-n'."

## 4. Register what you started
For every session you actually OPENED or QUEUED, call `start_session` with `spec`, `spawner: "claude-session"`, `externalRef` set to the new session's id or URL, and `branch`. An unregistered session is invisible to `fleet_status`, which is the one read that answers "what are my agents doing" — and an invisible fleet is worse than a list I paste myself. Skip this for tier 3: I have not opened anything yet.

## Pending proposals
A pending proposal is not work yet — accepting it is my call, not yours. Do NOT call `accept_proposal` or `decide_proposals` on your own. Tell me which pending findings look parallel-safe and offer to accept and launch them in one step; if I say go, use `decide_proposals` for the batch.
Then WAIT before launching those. Accepting creates the spec immediately but its body is written afterwards by a queued run, so for a minute or so the spec is just the proposal's one-line summary. `plan_batch` will not offer a spec that is still drafting, so re-run it until they appear rather than launching straight from the accept — a session started too early gets the summary instead of the spec.

Aim for the largest safe parallel batch. Each spec completes on its own when its PR merges (Drift) — nothing to mark done. Finish by telling me what is running, what is waiting, and what I still have to decide.
