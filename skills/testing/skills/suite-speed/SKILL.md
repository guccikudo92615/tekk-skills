---
name: suite-speed
description: 'Audits the test suite for wall-clock it does not need to spend — real network, database, filesystem or timer I/O that should be faked, heavy per-test setup that could be shared, oversized fixtures, serial-but-independent cases, redundant assertions and over-broad snapshots — and proposes rewrites that keep every check. Use when test files or runner config changed, or the suite has never been reviewed for speed.'
---

# Skill: suite-speed

**Skill:** the wall-clock. Rank what is in front of you by how long it plausibly
takes, then attack the top of that list. One test doing real network I/O in a
loop dwarfs fifty micro-inefficiencies, and a rewrite that saves nothing
measurable is not worth a reviewer's attention.

## What to evaluate

1. **Real I/O that should be a fake.** A live HTTP call, a real database, the
   filesystem, or a real timer waiting out a delay. Almost always the dominant
   cost. Replace with a fake, an in-memory double, or fake timers — **unless the
   round trip is the point**: an integration or contract test exists precisely to
   exercise the real thing, and faking it does not make it fast, it makes it
   pointless.
2. **Heavy or repeated setup.** Per-test work that could hoist to a shared
   `beforeAll`, a module-level fixture, or a built-once container/app instance —
   a server booted per test, a schema migrated per test, a bundle compiled per
   test. The constraint is isolation: setup that produces MUTABLE state the tests
   write to must stay per-test, or you have traded a slow suite for a flaky one.
3. **Oversized fixtures.** A 5MB JSON, a full production seed, a hundred-row
   factory where a three-field object exercises the same branch. Shrink to the
   minimum that still covers what the test asserts.
4. **Serial but independent cases.** Files or cases that share no state yet run
   one at a time because the runner was configured for the one suite that does
   share state. Enable the parallelism where independence actually holds, and say
   how you established it.
5. **Redundant assertions and over-broad snapshots.** The same property checked
   five ways, or a whole-page snapshot where one targeted assertion is cheaper,
   more stable, and says what it means. Trimming these speeds the suite and
   improves it — a giant snapshot is not a check, it is a diff.
6. **Waits that cost time on purpose.** A polled retry with a generous interval,
   a "let it settle" sleep, a timeout that every passing run still spends. Waiting
   on the condition is both faster and more honest than waiting on a duration.

## The invariant — behaviour and coverage preserved

Every rewrite must carry an explicit proof, or it is out of scope:

- **Same assertions.** It checks exactly what it checked before. Faster because
  it asserts less is forbidden.
- **Same coverage.** No branch, line or case that was exercised stops being
  exercised.
- **Same defect detection.** It still fails on the same bugs. Faking a
  dependency is valid only if exercising the real one was never the point.
- **State the proof in the proposal**: "identical `expect(...)` set; the same
  three branches hit; the fake returns the shape the real call returned."

A rewrite that cannot show this is not a trim you can propose. If the only way
to make a test faster is to test less, report it as slow-but-load-bearing and
leave it alone.

## How to verify before you claim

- **Estimate the saving and show the arithmetic.** "~40 real HTTP calls at
  ~150ms each → ~6s"; "boots the app 120 times at ~400ms → ~48s, once would be
  ~0.4s". An unquantified speedup is an opinion.
- **Check the runner config before rewriting a test.** Worker count, `maxWorkers`
  in CI, `testTimeout`, coverage instrumentation left on locally, a global setup
  that runs per file — suite-level settings often explain more of the wall-clock
  than any single test does.
- **Confirm independence before parallelizing.** Shared database rows, a fixed
  port, a temp path, a module-level cache: any of them makes concurrent
  execution a source of intermittency, which is a worse problem than the one you
  are solving.
