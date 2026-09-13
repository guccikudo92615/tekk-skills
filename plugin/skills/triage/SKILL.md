---
name: triage
description: 'Resolve the whole spec board against git reality in one pass — find specs that shipped but were never closed, were marked started but never begun, duplicate each other, or target code that no longer exists. Use when the board is untrusted ("what is actually done?", "clean up the specs"), before planning or dispatching from it, and after a batch of PRs merges. Operates on specs; the pending-proposal inbox is sweep.'
---

Triage — resolve the whole board against git in one pass. No arguments: checking specs one at a time is the cost this exists to remove.

A spec's status is a **claim, not a fact**. Nobody lies on purpose. An agent edits a body and the board calls it started. A branch name accidentally claims a closure. A finished spec stays open because ticking the checklist is a separate keystroke from merging. Each is invisible alone; together the board stops describing the work, and every plan drawn from it inherits the error.

## Ground rules

1. **Git is the truth; the board is a claim.** Never resolve a spec from its status, its title, or its drift verdict. Open the files and check.
2. **file:line or it did not happen.** Every verdict carries a locator someone else can check. "Looks done" is a guess wearing a conclusion's clothes.
3. **A stale path is not a dead spec.** A moved file reads exactly like a deleted one. Search by symbol before concluding anything is gone — closing a spec because someone moved a file discards a real, still-unfixed bug.
4. **Status corrections are yours. Killing work is the user's.** Moving a never-started spec out of in-progress is a factual repair; make it. Closing a spec as obsolete, duplicate or superseded ends someone's intent — propose it with your reasoning and wait.

## Order of work

Each step narrows the next.

1. **`get_workspace_overview`** — the drift block names specs whose status disagrees with what merged. Read it as a list of *candidates*, never verdicts, and read the misreporting section below first.
2. **`list_specs`** — the whole open board. Titles and dates only; do not fetch bodies yet.
3. **Cluster before you read.** Group by theme and by creation date. Specs filed within minutes of each other are usually one loop run's output and are the richest source of duplicates. A cluster costs one investigation and resolves several specs.
4. **Verify each cluster against code** in one batched pass — grep the claim, not the title.
5. **Act:** repair statuses, propose closures, record every verdict in the body it belongs to.

## What rots, and how to find it

**Shipped but never closed.** The spec describes behaviour that now exists. Grep the feature's distinguishing symbol, not its title, which will have drifted. The hard case is a spec whose implementation moved: one asked for a field on a particular module, and it shipped in a different module off a different table, so every search by the spec's own words missed it for months.

**Marked started, never started.** Zero checklist items ticked, no PR, drift saying not-started. Usually the automatic move described below, not a stalled human. Move it back to `todo` — a factual repair, no ceremony.

**Same bug, two specs.** Compare by the file and line each names, never by title: the same defect gets two unrelated-sounding names months apart. Keep the better-written one, propose closing the other, and record in the survivor's `## Out of scope` anything unique the closed one carried, so the intent survives the closure.

**One job, N specs.** Several loops independently proposing the same work produces sibling specs that each look reasonable. The tell: no pattern for the thing exists in the repo yet, so whoever ships first sets the convention for all of them — which makes it one PR by nature. Propose merging into the broadest, carrying across every specific file, line and correction from each.

**Premise gone.** The spec targets a model the product no longer has. Check the schema and the route, not the prose. Propose closing, and record what a fresh spec would need to be grounded against.

**Junk.** No problem statement, no build order, nothing buildable. Propose closing; a title is not a spec.

**Checkboxes nobody can tick.** Completion counts **every** `- [ ]` in the body, `## Acceptance` included. A box requiring a human decision, a waiting period, or work in someone else's dashboard blocks completion forever — a spec carrying only acceptance-criteria checkboxes can never close even when fully built. Convert acceptance to prose bullets; move human decisions into `## Decisions` as stated facts.

**A parent carrying checkboxes.** A parent spec's children are its checklist. A box in its own body can only ever produce a false verdict. Delete the box, state the tree in prose.

## How the board misreports

Recognise these or you will "fix" specs that were fine and trust verdicts that are wrong.

- **Editing a spec marks it started.** Updating a spec moves a `todo` spec to `in-progress` on a **description** edit. So your own pass creates the anomaly it reports. **Reset the status after every body edit.** Edits that pass no description (adding file scopes, retitling) do not trip it, so fill those in freely.
- **Drift misjudges parent specs, both ways.** A parent with no checkboxes reads as complete, so any historical closing PR makes it look done — even at zero children finished. Inversely, a parent carrying one stray box is judged on that box while its children are ignored. **Believe the child count, never the verdict.**
- **A branch name can claim a closure.** A spec id anywhere in a branch name can mark a PR as closing it, with no keyword and no way to retract after merge. A narrow repair branched with a spec's id in its name gets recorded as completing that whole spec.

## Closing a spec

The blessed path is a merged PR carrying `Closes TEK-n` over a finished checklist, and it belongs to the authoring agent, not to this pass.

Marking a spec completed by hand is ungated and exists for **reconciliation** — a spec whose work demonstrably shipped under someone else's PR. Every hand-close records in the body why the automatic path did not apply, with the locators proving the work exists. A close whose reasoning is not written down is indistinguishable next month from a mistake.

Write that record as a table of *what the spec asked for* → *where it lives now*, and name every deviation. A reader who only sees "completed" has to re-derive all of it from scratch.

## Gotchas

- **Creating a spec refuses a body without verified assumptions.** Each needs a real locator — a file and line, a URL, or a measurement with its source. "Checked the code" is rejected. Let the gate work; it catches wrong claims.
- **Only verified assumptions render.** A refuted claim vanishes from the body, so put what you *stopped* believing in `## Findings` or it is lost.
- **Fetching one spec is cheap; fetching thirty is not.** Cluster from the list, then fetch only what you will act on.
- **Other sessions are writing the same board.** Re-read the overview before reporting totals; specs appear mid-pass.

## Report

State the count before and after, then what changed and why, grouped by check. Separate the factual repairs you made from the closures you are proposing. Name what you examined and left open, and what you never reached — an unexamined stratum silently reads as a clean one.
