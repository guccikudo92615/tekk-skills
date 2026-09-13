---
name: architecture
description: 'Audits the module level — layering violations, new coupling and circular imports, logic duplicated instead of reused, responsibility creep, vendor specifics leaking through a seam, and files that grew into grab-bags. Use when reviewing services, libraries, core modules, barrels, adapters, or any change that moves logic across a boundary.'
---

# Skill: architecture

**Skill:** the shape of the code at the module level, and the refactor that
restores it. You are looking for structure that will make the *next* change
harder — not for structure you would have designed differently. Statement-level
cruft is the deslop skill; stale references to a renamed value are propagation's.

## What to evaluate

1. **Layering violations.** A change reaching across a boundary the codebase
   otherwise respects: a route doing raw data access, a UI component calling a
   service directly, business logic parked in a controller, a background job
   importing from the HTTP layer. The finding is the boundary — name the sibling
   that respects it.
2. **New coupling.** A module now reaching into another's internals rather than
   its interface; a circular import introduced (or one link away); a change that
   ripples across three files because concerns are entangled. Say what the cycle
   is and where to cut it.
3. **Duplicated logic.** The change re-implements something that already exists,
   creating two places to keep in sync. Search by NAME and by what it does
   (domain noun + verb) — the existing copy is often spelled differently. Merge
   only when inputs, side effects, error behaviour and return shape match.
4. **Responsibility creep.** A module or function grown well past one job; a new
   parameter that is really a second responsibility smuggled in (a boolean that
   switches the function's mode is the classic tell).
5. **Leaked abstractions.** Provider or vendor specifics escaping the seam meant
   to contain them — a raw SDK payload crossing an interface, a database-shaped
   row returned from a domain function, an HTTP status decided deep in a service.
6. **Grab-bag files.** A file that grew into five jobs, where every change is
   risky because everything lives together. Propose a split along an
   **empirically-discovered seam** — what actually clusters by call graph,
   co-change history, or shared state — never an aesthetic one, and only when
   the size genuinely hurts changeability.

## How to propose

- **Name the concrete move.** "Extract X into Y", "invert this dependency so A
  no longer imports B", "collapse these two into the existing helper". A
  finding that ends at "this is tangled" is not actionable.
- **Keep the public API.** A split or move must be isomorphic: every existing
  import path keeps working (a façade re-export is fine), behaviour identical,
  no compile or type drift. A restructure that breaks callers is out of scope
  here.
- **Size it to one reviewable change.** If the honest fix is a five-module
  reshape, propose the first cut that stands on its own and say what it sets up.

## How to verify before you claim

- **Read the sibling first.** Half of what looks like a violation is the house
  pattern. If three neighbours do it the same way, the pattern is the standard
  and your finding is a preference.
- **Prove the duplication.** Show both implementations and that their behaviour
  matches — a near-name is not evidence, and merging two things that differ in
  an error path is how a refactor becomes an outage.
- **Prove the cycle.** Name the import chain, file by file, rather than
  asserting entanglement.
