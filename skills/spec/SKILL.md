---
name: spec
description: 'Write a spec the board will accept and the drift gate can actually close — research the claims first, keep every checkbox finishable by the coding agent, and write acceptance as prose rather than boxes. Use when creating or rewriting a spec, turning a conversation into tracked work, or when a spec has stalled because nobody can tick its checklist.'
---

Write a spec that is implementation-ready and can actually reach `completed`. Two different things go wrong, and both are permanent once the spec exists: a body whose claims were guessed, and a checklist nobody can finish.

## Research before you write, not after

A spec that exists is implementation-ready, so its claims about the codebase must be **researched, not guessed and annotated afterwards**. Open the files. If the code contradicts the plan, the plan changes — that is the point of doing it first.

Every load-bearing claim goes in `assumptions` as `{ claim, verdict, evidence }`, where a load-bearing claim is one that changes the design if it is wrong. Evidence must be a **locator someone else can check**:

- `path/to/file.ts:123`, or a range like `:164-169`
- a URL, for an external fact
- a measurement with its source, e.g. `prod 2026-08-03: repo 5 has 1257 summaries vs 4 embedded files`

"Verified", "checked the code" and "confirmed" are rejected — once persisted they are indistinguishable from a guess. Delete any refuted claim from the body; do not leave it as an open question. An empty `assumptions` array means "I checked, nothing here is load-bearing", which is right for a genuinely trivial change and almost never right for a spec about existing code.

## Every checkbox must be finishable by the coding agent, in the PR

Completion counts **every** `- [ ]` in the body, `## Acceptance` included. A box nobody can tick is a spec that can never close. These are banned as checkboxes:

- **Human decisions** — "review with the team", "decide the shape", "get approval". Make the decision *before* writing and record it as a stated fact under `## Decisions`, or split it into its own spec. If a decision is still open, the spec is not ready.
- **Waiting periods** — "watch one week of runs", "monitor for false positives", "observe after deploy". An agent cannot wait a week. If it genuinely needs measuring later, that is a follow-up spec.
- **Out-of-band work** — anything done in a dashboard, another tool, or by another person.

Write `## Acceptance` as **prose bullets (`-`), never checkboxes**, so criteria describe how to verify without gating completion. The cautionary case is real: a spec with four boxes, two of them "review with the team" and "watch one week", sat unstarted for a month while its work ran in production.

## Shape

- `## Summary` — first and required. Two to four sentences of plain English: what changes, why, who for. Written for someone who was not in the session that produced it. No file paths, no line numbers, no unexplained shorthand. On a feature, open with a user story: as a *who*, I want *what*, so that *why*.
- `## Sources` — every external input, each with a locator. A source you cannot link must be **digested** here: enough of what it actually said that a reader never needs the original. Characterising it ("most of it we already do") records nothing.
- `## Problem / Context` — what is wrong or needed. This is where file:line grounding belongs.
- `## Findings` — what the session learned that is not a build step: what surprised you, what turned out false. Nothing else has room for it, so whatever you leave out is lost when the session ends.
- `## Decisions` — the options weighed, the one taken, and why each other was rejected, with the evidence that settled it. An unrecorded rejection gets re-proposed in six months.
- `## Build order` — an ordered checklist. Drift derives completion from it.
- `## Acceptance` — prose bullets. A test, a command, an observable outcome.
- `## Out of scope` — optional, but it is where a closed duplicate's unique intent survives.

## Carry what cannot be re-derived

The spec is the durable record of the session that produced it. Write it so that session could be deleted and nothing would be lost. Build steps are cheap to reconstruct from the code; the document you were handed, the option you rejected and why, and a measurement taken against production on a given day are not. Spend the length there, not on restating what the code already says.

## Closing it later

A merged PR carrying `Closes TEK-n` in its **body** is the only thing that closes a spec, and only over a finished checklist. Both halves are the authoring agent's job in the same PR: tick what you actually finished, and write the line.

A PR that merely *mentions* the id links it for display and moves nothing — deliberately, since a mention is often a cross-reference. A branch named after a spec links the PR and closes nothing.

Half-finished is fine and honest: tick what you did, still write `Closes`, and the spec lands as incomplete with the remainder visible.

## One feature, one spec tree

A small, indivisible change is a single standalone spec with its phases as an ordered `## Build order`. When a feature genuinely decomposes into independently-buildable parts, use a parent with sub-tasks, each carrying its own PR and file scope; the parent completes when its children do. Group several *independent* initiatives under an epic — never the phases of one feature.

Declare `touches` — the file globs the work will change. It costs one field and it is what lets the board notice later that a spec's every declared path has changed in git while the spec sat open.

## Gotchas

- **Updating a spec replaces the whole description.** Retyping someone else's body to change one clause risks corrupting the record you are trying to protect. Prefer a targeted edit, and where you must retype, keep the original text to hand.
- **A description edit marks a `todo` spec as started.** Reset the status afterwards if that was not what you meant. Passing no description (adding `touches`, retitling) does not trip it.
- **Only verified assumptions render in the body.** Put what you stopped believing in `## Findings`, or it disappears.
