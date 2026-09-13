---
name: deslop
description: 'Audits the statement level for AI-generated cruft — pass-through wrappers, single-use helpers that earn nothing, dead and speculative code, comments restating the code, nested ternaries, near-duplicate logic, and markup buried under redundant wrappers. Use when reviewing ordinary code churn that no sharper skill claims.'
---

# Skill: deslop

**Skill:** behaviour-preserving simplification of the code that changed — make it
clearer and leaner without changing what it does. You prize readable, explicit
code over clever compression; holding that balance IS the skill. Module-level
structure is the architecture skill; stale references to a renamed value are
propagation's.

The proof obligation in the loop file applies to every item below: you produce
the isomorphism argument *before* you propose the delete.

## What to hunt

1. **Needless indirection.** Pass-through wrappers, single-branch helpers, and
   layers that exist only to forward arguments. Collapse them.
2. **Single-use helpers that earn nothing.** A function or constant used at
   exactly one call site whose name adds no meaning. Inline it — and keep it
   when the name is what makes the call site readable. That judgment is the
   whole rule.
3. **Dead and speculative code.** Unreachable branches, unused exports, options
   and parameters nothing passes, scaffolding from an earlier iteration, a flag
   with one value. Delete it — and check for the *string* references a
   type-checker won't see before you do.
4. **Comments restating the code.** Remove them. Keep the ones that explain
   *why*, a non-obvious behaviour, or a library quirk — those are the comments
   this codebase asks for, and deleting one is a real loss.
5. **Nested ternaries and dense one-liners.** Prefer an `if`/`else` chain or a
   `switch`. Explicit beats compact.
6. **Near-duplicate logic.** Something is implemented twice. Search by the
   helper's name AND by what it does (domain noun + verb). Propose reuse — but
   merge only symbols with the same inputs, side effects, error behaviour and
   return shape. A similar name is not proof.
   **A copy that has DIVERGED is still your find — the drift is the harm.**
   Identical copies cost a second edit; a drifted one costs a difference nobody
   chose and nobody can see. Never let "they are not identical" be the reason
   you leave one out.
   But mind the line the loop draws. Where the copies differ only in form —
   naming, ordering that nothing observes, dead wrapping — the merge preserves
   behaviour and is yours to propose. Where they differ in what a user or caller
   would SEE — wording, styling, spacing, elements, an error path — then picking
   a winner changes the losing call site, and that is a behaviour change: **report
   the drift, name every way the copies differ, and say plainly that resolving it
   is a product decision you are not making.** Do not pick the winner yourself.
   That is the loop's law, not a nicety: an "obviously better" behaviour change
   is another loop's finding, not yours.
7. **Wrapper soup in markup.** Elements buried under redundant wrapper `div`s or
   fragments that add no layout or semantics. Flatten to the minimal tree — and
   check nothing depends on those nodes (a selector, a test, a style rule) first.
8. **Defensive noise.** A null check on a value the types say cannot be null, a
   `try`/`catch` that rethrows unchanged, a default that the caller always
   supplies. Each one costs a reader attention and buys nothing.

## How to verify before you claim

- **Show the twin.** For a duplicate merge, both implementations side by side
  and the behaviours that match. This is the item most likely to be wrong.
  When they are NOT identical, show every difference — and say which of them are
  form and which are behaviour. A merge presented as a pure move, that in fact
  restyles a view, is a regression shipped as a cleanup.
- **Show there are no callers.** For a delete, the search that establishes it —
  including string/dynamic references, config, and anything crossing a package
  boundary. When your brief names other codebases as readable, that boundary is
  reachable: call `list_codebases` and search them too before claiming absence.
  A library's consumers are where a "dead" symbol usually turns out to be alive,
  and they are the one place this repository cannot tell you about. If you could
  not check them, say which repositories your search covered — an unqualified
  "no callers" reads as "none anywhere", which is a claim you did not make.
- **Prefer the change that removes a concept, not just characters.** Deleting a
  wrapper that nothing needed is worth more than compressing a readable block.
- **Consolidating a duplicate is the ONE case where you read outside the diff.**
  Otherwise, changed code only. On a run with no diff you pick your own surface
  to audit (no worklist is handed to you — the coverage store that used to supply
  one is gone), and the same rule applies from wherever you start:
  having found one duplicated symbol in a file, sweep that file for the others
  before you write. The copies travel together, and reporting one while missing
  its neighbour makes the fix look smaller than it is.
