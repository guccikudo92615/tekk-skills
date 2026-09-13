---
name: context-engineering
description: 'Audits what fills a product''s model context and whether any of it is dead weight or dead ends — memory written but never read, prompts demanding facts nothing supplies, blocks that grew past their usefulness. Use when reviewing prompts, memory, summarisation, compaction, or any injected context block.'
---

# Skill: context-engineering

**Skill:** what fills this product's model context, and whether any of it is
dead weight or dead ends. The failures in this lane are chronic and silent — a
writer with no reader, a prompt demanding a fact nothing supplies, a summary of
a summary feeding a decision. None of them crash or log; they quietly make the
agent worse while every component looks healthy. You will not find them by
reading top-to-bottom. You find them by asking, of each piece of context
machinery: **who actually consumes this, and does it change what they do?**

## What to hunt

1. **Written, never read.** For every memory artifact (a summary table, a notes
   file, a "history" column, a cache of past outputs): list every writer and
   every reader, with `file:line` on both sides. A writer without a matching
   reader isn't future-proofing — it's spend plus a false belief that the
   system remembers. Subtler forms: written fully but read as a head-slice that
   structurally never reaches the valuable section; or read only into display
   copy, steering no decision.
2. **Lossy compression chains.** Trace every derived artifact to its sources
   and multiply the truncations. A 1,200-char excerpt of a 16,000-char document
   is 7.5% — each cap can look reasonable while the chain preserves almost
   nothing. Derived-from-derived is the red flag, and what matters is what the
   *final* consumer sees after every stage's cut.
3. **Prompt assembly census.** Enumerate every block injected into each model
   call: source, measured size (read the real constants — comments lie), and
   whether it is **data or exhortation**. Sum the static instruction prose
   separately; ask of each block: *if this were deleted, would tonight's output
   change?* Volatile content interleaved into a cacheable prefix is also a
   finding — it silently breaks continuity between calls.
4. **Memory that fails its one job.** Name each memory's single load-bearing
   purpose ("never re-ask the user this", "know what's been processed"), then
   find the mechanism that fulfills it. A dedup key that hashes wording lets
   every rephrase through; a coverage store with no expiry means seen-once is
   seen-forever. Test the mechanism against one concrete case the memory
   should have caught, traced through the code.
5. **Structurally-empty inputs and unsatisfiable instructions.** A prompt that
   names a source which is always empty at runtime ("ACTIVE GOALS" that no code
   path populates), or demands a fact nothing in context supplies ("size to
   current scale" with no number anywhere). An instruction the model cannot
   satisfy from its context is an instruction to hallucinate — either supply
   the fact or make its absence an explicit stated line.
6. **Sprawl, dead gates, severed judgment.** Count model calls per unit of work
   and ask what each can change about the outcome. A review stage that cannot
   say no is paid annotation, not a gate. A stage that judges but cannot
   investigate — fed only truncated strings from a stage that is gone by
   judgment time — cannot course-correct; whatever must redirect the work
   mid-run has to live in the session that can do the work.
7. **Context rot.** Unbounded history growth with no compaction; stale or
   irrelevant blocks crowding out what the model needs; the same content
   injected twice through different paths. This is the *reasoning* cost of the
   window — the spend side is the `llm-spend` skill on this same shelf.

## The archetype run (worked example)

The workspace's agent has a system prompt that has grown by accretion. You
census it (skill 3): 14KB of standing text, half of it rules patched in over
months, several sections whose subject no longer exists in the code, two
blocks saying almost the same thing in different words. You check the read
side: nothing consumes three of the sections. You then check practice —
`WebSearch` for current system-prompt guidance (identity + capabilities +
workflow, lean standing core, task-specific detail loaded on demand) and
`WebFetch` the SDK's own docs where behavior is load-bearing. Your proposal:
the specific restructure — which sections to delete (with proof nothing reads
them), which to merge, what moves out of the standing prompt into on-demand
context — with the before/after byte counts and the source you checked
practice against. That is this skill at full strength: measured census +
dead-weight proof + current practice → one concrete, defensible restructure.

## How to verify

Every claim in this lane is a pair of locators: the **injection/write site**
and the **read site** (or the demonstrated absence of one), both `file:line`.
Sizes are measured, not estimated — find the actual constants (`slice(0,`,
`MAX_`, `_CAP`, truncation helpers) and do the division. "This block is
probably too big" is not a finding; "8KB injected, consumer reads 400 chars"
is. Prefer the deterministic fix: rendering the source table beats maintaining
a prose summary of it; a mechanical check beats a model run spent on
bookkeeping.

## Discipline

- The sharpest deliverable here is often a **deletion** — of an unread
  artifact, an unused block, a stage that changes nothing. Frame it as
  capability: a leaner window reasons better.
- Don't repair an organ the body doesn't use: if the read side barely exists,
  fixing the write side preserves spend without adding value. Propose delete,
  and rebuild only when a real reader appears.
