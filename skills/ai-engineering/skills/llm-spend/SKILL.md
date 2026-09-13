---
name: llm-spend
description: 'Audits what the product pays to run its LLM calls — agent turn tax, context bloat, model-escalation ladders, missing prompt caching, an oversized model for a bounded task, unbatched bulk calls, retries with no ceiling, and tool surfaces paid for in every cached prefix. Use when reviewing an agent loop, a model call site, token/usage accounting, or any change whose question is "same output, less money".'
---

# Skill: llm-spend

**Skill:** the bill for the same output. Every model call in front of you, asked:
*could this produce the identical result for less money?* Not "is it good" —
that is every other skill here. If the cheaper form would be worse, there is no
finding: a saving bought with quality is a regression wearing a discount.

## What to evaluate

1. **Turn tax — check this FIRST on any agent loop.** In a multi-turn session
   every turn re-submits the accreted context (history + prior tool outputs +
   tool definitions), so cost grows **quadratically in turns, not linearly**:
   over n steps it is `n·s₀ + p·n(n−1)/2` = Θ(n²), the state snowball. A task
   taking 20 tool calls costs roughly **200×** a single call, not 20×. This is
   why turn count dominates payload size — a lean 20K context read across 8
   turns costs far more than a fat 100K context read once.

   The tell is a **synthesis stage carrying retrieval affordances**: a stage
   whose inputs are already assembled inline (drafting, authoring, reviewing,
   summarising, judging) that is nonetheless handed tools, subagent spawn, or a
   high `maxTurns`. Detect it structurally rather than by recognising a
   remembered example — for each stage write down, independently, (a) its
   **declared job** from its prompt/name/docs and (b) its **granted affordances**
   from its config: tool allow-list, registered MCP servers, `maxTurns`, whether
   it can spawn subagents. Only where (b) EXCEEDS (a) is there a finding. A stage
   that looks *under*-provisioned is the opposite finding and belongs to
   `harness` — adding capability costs money on purpose. Both halves are static
   facts in the code: read the allow-list and the turn budget, do not infer them.

   Propose matching affordances to the stage's real shape — retrieval stages
   (job = gather) get tools, many turns, a cheap model; synthesis stages (job =
   reason over what is given) get no tools, or one bounded read-only
   spot-check, and `maxTurns` ≈ 1–2. Quantify as **turns × per-turn context**,
   and name the tools you would remove.
2. **Context bloat (static payload).** A call carrying far more context than the
   task uses — a whole vault, repo or transcript stuffed in every tick.
   **Measure the assembled payload before blaming it.** This is the most common
   misdiagnosis in agent cost work: a huge cache-read figure looks like a bloated
   prompt, but a multi-turn session re-reading a *small* context produces the
   same number. If the per-call payload is already lean, the spend is upstream in
   turns and trimming the payload buys nothing.
3. **Model-escalation ladders.** A retry or fallback path that escalates to a
   pricier tier on failure (cheap → cheap → frontier). Every escalation silently
   multiplies the bill, so an unreliable cheap stage quietly bills like a
   frontier one. Read the escalation trigger; propose fixing the underlying
   failure or capping the ladder rather than paying it — and note that a
   frequently-escalating stage is also a quality signal.
4. **No compaction, unbounded window growth.** The direct mitigation for turn tax
   when the turns are genuinely needed: a long session that never summarises,
   prunes or windows its history, so every later turn drags the whole transcript.
   A rolling summary, a bounded window, or dropping tool outputs once consumed
   typically cuts token volume by half or more.
5. **Missing prompt caching.** A large, stable prefix — system prompt, tool
   definitions, fixed context — re-sent uncached on every call, where cached
   reads cost roughly a tenth. **Hit rate is the metric**, so check what mutates
   the prefix: a timestamp, a reordered tool list, or per-user text spliced into
   the stable region defeats the cache. Moving mutable content *after* the cache
   boundary is often a bigger win than enabling caching at all.
6. **Oversized model for a bounded task.** A top-tier model on classification,
   extraction, or a short structured call where a cheaper tier returns the same
   output. Propose the downgrade only where quality is genuinely not at risk —
   if it is a judgment call, it belongs to `harness`. Where a mixed workload runs
   entirely on one frontier model, routing the easy majority down is the classic
   win (published routers hold most of frontier quality while sending a small
   minority of calls to the strong model).
7. **No batching.** Many independent calls that a batch or async API prices lower
   — bulk tagging, backfills, report generation, offline enrichment. Never
   propose it for an interactive path; the latency is the product there.
8. **Retries with no ceiling, and oversized tool surfaces.** A retry loop that
   re-spends full tokens on every attempt with no cap; and tool schemas sitting
   in the cached prefix of every call for a stage that never invokes them.

## How to verify before you claim

- **Audit the pipeline stage by stage before proposing anything.** For each
  stage list its job (retrieve vs synthesise), model, `maxTurns`, tool
  allow-list, and retry path. The mismatches fall straight out of that table,
  and it stops you proposing a payload trim on a stage whose payload is small. A
  stage that is cheap per call but runs many turns is usually the most expensive
  thing in the system.
- **Read turn counts, do not infer them.** Cite `maxTurns` and the tool surface
  that makes turns possible, or a telemetry span. Arithmetic on token totals is a
  hypothesis — say so if that is all you have.
- **Show the waste in the code.** For "no caching", show the stable prefix
  re-sent uncached. For "context bloat", show what is assembled versus what the
  output uses. A claim the config contradicts is not a finding.
- **Name the magnitude honestly, and do not invent dollars.** "A synthesis stage
  with six tools and `maxTurns: 100`, so the context is re-read once per turn" is
  a sized claim. Multiplying by an assumed request volume to manufacture a
  "$X/day" headline is not — give the per-call or per-tick saving and stop.
- **Same output is the constraint.** State what stays identical: the same
  assertions from the eval, the same structured fields, the same behaviour on
  the paths you checked.
