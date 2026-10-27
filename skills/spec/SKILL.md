---
name: spec
description: 'Write a spec the board will accept and the drift gate can actually close — research the claims first, keep every checkbox finishable by the coding agent, and write acceptance as prose rather than boxes. Use when creating or rewriting a spec, turning a conversation into tracked work, or when a spec has stalled because nobody can tick its checklist.'
allowed-tools:
  - mcp__tekk__list_specs
  - mcp__plugin_tekk_tekk__list_specs
  - mcp__tekk__get_spec
  - mcp__plugin_tekk_tekk__get_spec
  - mcp__tekk__create_spec
  - mcp__plugin_tekk_tekk__create_spec
  - mcp__tekk__update_spec
  - mcp__plugin_tekk_tekk__update_spec
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

The board enforces this: a checkbox that reads as a human decision, a waiting period, or work after the merge is rejected with the offending line named, and so is any checkbox under `## Acceptance`. Write `## Acceptance` as **prose bullets (`-`), never checkboxes**, so criteria describe how to verify without gating completion. The cautionary case is real: a spec with four boxes, two of them "review with the team" and "watch one week", sat unstarted for a month while its work ran in production.

## Shape

- `## Summary` — first and required. Two to four sentences of plain English: what changes, why, who for. Written for someone who was not in the session that produced it. No file paths, no line numbers, no unexplained shorthand. On a feature, open with a user story: as a *who*, I want *what*, so that *why*.
- `## Why` — required, right after the Summary. The purpose, for someone who has never seen the codebase: who is hurt or blocked today and how, what becomes possible once it ships, and what happens if it is never built. The Summary says what changes; this says what it is *for*. A coding agent reads it to choose between two ways of building when the steps run out. No file paths here, and it must stand alone — never "see the parent spec". The board rejects a body without one, and a Why that is only a pointer.
- `## Sources` — every external input, each with a locator. A source you cannot link must be **digested** here: enough of what it actually said that a reader never needs the original. Characterising it ("most of it we already do") records nothing.
- `## Problem / Context` — what is wrong or needed in the code today. This is where file:line grounding belongs. It is evidence for the Why, not a replacement for it.
- `## Findings` — what the session learned that is not a build step: what surprised you, what turned out false. Nothing else has room for it, so whatever you leave out is lost when the session ends.
- `## Decisions` — the options weighed, the one taken, and why each other was rejected, with the evidence that settled it. An unrecorded rejection gets re-proposed in six months.
- `## Build order` — an ordered checklist. Drift derives completion from it.
- `## Acceptance` — prose bullets. A test, a command, an observable outcome.
- `## Out of scope` — optional, but it is where a closed duplicate's unique intent survives.

## Pictures

Writing is the cheap part of a session; reading the code was the expensive part. So be generous with a clear Summary and, when it helps, add **one picture**. Tekk draws certain fenced blocks in the body; agents still read the plain text. Put the picture right under the Summary, where the first screen of the spec is read. Pick it by what the work is:

- **Something reaches what it should not** (data exposure, a missing gate) → ` ```path ` with `{"steps":[{"label","detail","ref"}],"cutAfter":<index of the step after which the fix cuts>,"cutLabel","end":"danger"}`. A step may be a plain string.
- **What lives where in one large file** → ` ```filemap ` with `{"total","unit":"lines","segments":[{"label","from","to","tone":"hot|done|muted"}]}`.
- **A measured improvement** (speed, cost, size) → ` ```beforeafter ` with `{"rows":[{"label","now","target","unit"}]}`.
- **How modules or steps connect** → a small ` ```mermaid ` flowchart.
- **Numbers that are the argument** → ` ```stats ` with `{"items":[{"value","label","source"}]}`, at most 4. This may sit beside the one picture.

Rules: at most one picture and one stats row per spec. Every number in a picture also appears in `## Sources` or `## Problem / Context` with its locator. No picture is better than a decorative one. A picture that describes how things are today goes stale like any other fact; label it with what it measured. A fence the board cannot draw is returned as a warning naming it, and shows as plain code until fixed. When a spec is built from a proposal, copy the proposal's picture across as written rather than drawing a new one.

## Carry what cannot be re-derived

The spec is the durable record of the session that produced it. Write it so that session could be deleted and nothing would be lost. Build steps are cheap to reconstruct from the code; the document you were handed, the option you rejected and why, and a measurement taken against production on a given day are not. Spend the length there, not on restating what the code already says.

## Closing it later

A merged PR carrying `Closes TEK-n` in its **body** is the only thing that closes a spec, and only over a finished checklist. Both halves are the authoring agent's job in the same PR: tick what you actually finished, and write the line.

A PR that merely *mentions* the id links it for display and moves nothing — deliberately, since a mention is often a cross-reference. A branch named after a spec links the PR and closes nothing.

Half-finished is fine and honest: tick what you did, still write `Closes`, and the spec lands as incomplete with the remainder visible.

## One feature, one spec tree

A small, indivisible change is a single standalone spec with its phases as an ordered `## Build order`. When a feature genuinely decomposes into independently-buildable parts, use a parent with sub-tasks, each carrying its own PR and file scope; the parent completes when its children do. Group several *independent* initiatives under an epic — never the phases of one feature.

Declare `touches` — the file globs the work will change. It costs one field and it is what lets the board notice later that a spec's every declared path has changed in git while the spec sat open.

## Facts that go stale

Do not write a sequence number as a fact: "the next decision record is 0026", "the latest migration is 0180". It is true the day you write it and wrong once anything else lands, and a spec sits for days before it is built. Say "the next free number when you build this" and name the directory to list. The board warns when it sees the pattern.

The same goes for a figure your Why leans on. Re-derive it from your own Sources just before you submit: a headline cost taken from an old model alias, when the current one is a third of it, weakens the case more than having no number.

When a child's decision changes something its parent states, such as a count of tools or a default, update the parent in the same pass. Parent and children that disagree send the next agent to build two different things.

## Say why, and say it where the reader will look

The failure this section prevents is a body made entirely of implementation: components, tables, call sites, and no sentence about who it is for. Every detail in it can be right and a coding agent still cannot tell which of two reasonable builds serves the purpose, because the purpose is not written down anywhere. It is not in the repository either, so when the session ends it is gone.

Before you submit, read the Summary and the Why as the person who will build this next week and has none of your context. If they could not say what the work is for, rewrite them.

A **sub-task** is the spec most likely to be opened without its parent, by an agent dispatched to build exactly that piece. Its Why restates the parent's purpose in its own words, says what this part unlocks, and says what breaks if it is skipped. "Design decisions are in the parent" is not a Why.

Editing is held to the same style. An edit that deletes the Why, the Summary or the build order is refused; a gap the spec already had comes back as a warning, so ticking a box on an older spec still works.

## Gotchas

- **Updating a spec replaces the whole description.** Retyping someone else's body to change one clause risks corrupting the record you are trying to protect. Prefer a targeted edit, and where you must retype, keep the original text to hand.
- **A description edit marks a `todo` spec as started.** Reset the status afterwards if that was not what you meant. Passing no description (adding `touches`, retitling) does not trip it.
- **Only verified assumptions render in the body.** Put what you stopped believing in `## Findings`, or it disappears.
