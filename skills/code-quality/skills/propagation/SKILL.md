---
name: propagation
description: 'Audits what a rename left behind — every place still using the old form of a value that changed identity: env vars, exported constants and enum members, config keys, API routes, and the docs that cite them. Use when a diff renames or replaces a value, or when reviewing config, constants, env files or documentation.'
---

# Skill: propagation

**Skill:** the straggler. When a value changes identity — `X` becomes `Y` — you
find *every other place still using `X`* across code, config and docs, and
propose one diff that updates them all. The changed value is the join key; the
artifact is a complete, atomic sweep.

This skill is quiet most of the time. When no rename happened, say so and stop —
that is the correct outcome, not a failed run.

When your brief names other codebases as readable, the stragglers may be in
them — an API route the frontend still calls by its old path, a constant a
sibling service imports. Call `list_codebases` and sweep those too; a rename is
only complete when nothing anywhere still spells it the old way, and this
repository cannot see the half of that which lives elsewhere.

## What to evaluate

Read the change and identify a value that **changed identity** — the same
concept now spelled or located differently:

1. **Environment variables and secret keys** — `STRIPE_KEY` → `STRIPE_SECRET_KEY`.
   Stragglers: other `process.env.X` reads, `.env.example`, deploy config, CI
   workflow files, docs.
2. **Exported constants, enum members, types and functions** — a renamed export,
   a changed enum value, a renamed type. Stragglers: importers, re-exports, and
   *string* references to the old name (a persisted value, a switch case, a
   stored row).
3. **Config keys and defaults** — a restructured key, or a changed default that
   other code hardcodes to the old value.
4. **API routes and endpoint paths** — `/v1/x` → `/v2/x`. Stragglers: clients,
   tests, other handlers, and any docs or SDK snippet citing the old path.
5. **Docs and comments** — prose that names the old value. A README or ADR
   citing a removed route is a real straggler; a stale comment beside the code
   it describes is one too.

Then search the WHOLE repo — code, config, docs — for surviving references to
the old form.

## The guardrail — genuine deprecation only (load-bearing)

A wrong "fix" that rewrites a coincidental match is the failure this skill must
never have. Before flagging any reference, prove all three, or leave it:

1. **The old value is replaced, not coexisting.** Confirm `X` was renamed or
   removed — not that `X` and `Y` now both legitimately exist (a new option
   beside an old one is not a deprecation). If the old value still has a valid
   meaning, there is no straggler.
2. **The reference is the same semantic value, not a coincidental string.**
   `STRIPE_KEY` the env var is not the substring `stripe_key` in an unrelated
   URL or another subsystem's variable. Match on identity — scope, type, role —
   never fuzzy text. A grep hit is a candidate, not a finding.
3. **Updating it is correct.** The reference should point at `Y` now, and
   updating it preserves behaviour while fixing a latent break. If updating
   would change behaviour, or the reference intentionally pins the old value
   (a migration, a compatibility shim, a historical record), it is out of scope.

## How to propose

- **One diff, every straggler, consistent.** The value here is completeness:
  *"`X` was renamed to `Y` in `<file>`; these N references still use `X` —
  here they all are."* A partial sweep leaves new divergence behind, which is
  worse than the original.
- **List every reference with `file:line`** and show it is the same value.
- **Split when confidence differs.** Propose the references you can prove, and
  name the ambiguous ones separately for a human to judge rather than folding
  them in silently.
- **A reference that needs a LOGIC change, not just the new name, is not part of
  this sweep.** Flag it; leaving it in turns a mechanical update into a
  behaviour change.
