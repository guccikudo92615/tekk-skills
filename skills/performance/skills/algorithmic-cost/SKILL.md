---
name: algorithmic-cost
description: 'Audits the code itself for work that scales badly — a new nested loop turning O(n) into O(n²), a lookup that scans a list where a map would index it, repeated pure computation that should be memoized, a whole collection loaded to count or filter it in app, and allocation or copying that could be hoisted. Use for ordinary modules, transforms and utilities with no database or route in sight.'
---

# Skill: algorithmic-cost

**Skill:** the shape of the computation. Take the code in front of you and ask
what happens when its inputs are ten times bigger — not whether it is elegant,
but whether the time it takes grows in step with the data or ahead of it.

This is the skill for ordinary code: the transform, the merge, the grouping, the
formatter. The cost is often invisible locally and appears only when a real
workspace's data arrives.

## What to evaluate

1. **Nested iteration over the same data.** A loop inside a loop, or a `find` /
   `includes` / `indexOf` inside a loop, turning a linear pass into a quadratic
   one. The fix is nearly always an index built once: a `Map` or `Set` keyed by
   what the inner search looks up. State the before and after: "O(n·m) with n =
   proposals, m = specs → one pass to build the map, then O(n)".
2. **Repeated pure computation.** The same derivation recomputed per item, per
   render or per call — a parsed config, a compiled regex, a sorted copy, a
   formatter instance. Hoist it out of the loop or memoize it on a stable key.
3. **Count and filter in the wrong place.** Loading a whole collection to count,
   sum or check existence, where the source could answer directly. Related: a
   sort of the full set to take the first few, where a partial selection does.
4. **Copy and allocation churn.** Spreading an accumulator inside a loop (which
   rebuilds it every iteration), string concatenation in a tight loop, cloning a
   large structure to change one field, building an intermediate array for each
   step of a chain over a big collection.
5. **Work that scales with the wrong thing.** A pass over every row to answer a
   question about one, an operation whose cost tracks total data rather than the
   delta that changed, a recursive walk with no memo on a shared subtree.

## How to verify before you claim

- **Establish n from the code, not from imagination.** Trace what feeds the
  collection and what bounds it. If n is a handful of config entries, a
  quadratic loop is not a finding — say so and move on. The finding requires a
  path where n grows with users, rows or items.
- **Count the passes and name the complexity in the proposal.** "Two nested
  loops over the same array → O(n²) where n is the workspace's specs; a Map
  keyed by id makes it one pass" is a finding. "This could be optimized" is not.
- **Check the language's actual behaviour before claiming allocation cost.**
  Engines optimize a lot of what looks wasteful, and a micro-optimization with
  no measurable effect is noise. Prefer the change that moves a complexity
  class, not a constant factor.
- **Preserve the semantics exactly.** Same output, same order, same tie-breaks,
  same treatment of duplicates and empty input. An index-based rewrite that
  silently dedupes keys is a behaviour change, not a speedup.
