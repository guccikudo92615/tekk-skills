---
name: flakiness
description: 'Audits tests for non-determinism — fixed sleeps instead of waits on a condition, state shared across tests, order dependence, real clocks and timezones, unseeded randomness, real network or filesystem I/O, races and leaked handles — and proposes a stabilization or an honest quarantine. Use when reviewing e2e, integration or shared fixture/setup code, or a suite that flaps.'
---

# Skill: flakiness

**Skill:** the second run. Every test in front of you, asked: *if this ran again
right now — on a loaded machine, in a different order, in another timezone —
would it give the same answer?* A suite that answers "usually" is corrosive: it
teaches everyone to re-run red, and the real failure hides in the noise.

## The distinction that defines the skill

**Flaky is not broken.** A test that fails *deterministically* because a real
change altered behaviour is a genuine break, and the bug is in the product, not
the test — route it to the loop that owns that code and propose nothing here. A
flake is a test whose result varies without the code varying, and the bug is in
the test or its environment.

You cannot observe a pass-then-fail on the same commit from the repo alone. So
claim a flake only when the test *code* carries the reason it flaps — one of the
signatures below, named at `file:line`. Without one, treat it as a probable real
break and say so.

## What to evaluate

1. **Waits that are not waits.** A fixed `sleep(200)`, an arbitrary timeout, an
   un-awaited promise, an assertion that races the async work it checks. Under
   load the work is not done yet and the test fails for no reason. Fix: wait on
   the actual condition (the element, the row, the resolved promise), not on a
   duration.
2. **Shared and global state.** A module-level variable, a database row, a
   temp file or an environment mutation that one test writes and another reads.
   The tell is a test that passes alone and fails in the suite, or only in a
   particular order. Fix: isolate per test, reset between them, or remove the
   coupling entirely — never by pinning the execution order, which just hides
   the dependency.
3. **Time and timezone.** Real `Date.now()`, real timers, dates near midnight,
   month ends, DST transitions, or an assertion on elapsed wall-clock. Fix: fake
   timers or an injected clock, and pick fixture dates that are not boundaries.
4. **Unseeded randomness.** `Math.random`, UUIDs, faker-style generators, or a
   set/map iteration whose order is not guaranteed, feeding an assertion. Fix:
   seed it or assert on the property rather than the value.
5. **Real I/O.** A live network call, a third-party sandbox, DNS, a shared port,
   a filesystem path two workers can both claim. These fail on someone else's
   outage and on parallelism. Fix: fake the dependency, or make the resource
   unique per worker.
6. **Races and ordering inside the test.** Parallel operations with no
   deterministic completion order, an event assumed to arrive before the
   assertion, a check-then-act across an `await`. Name the interleaving that
   makes it flap — that naming is the evidence.
7. **Leaks across tests.** An open handle, timer, subscription or listener from
   a previous test firing during a later one, and the "worker process failed to
   exit gracefully" warning that comes with it. Fix: tear it down where it was
   created.

## The fix — stabilize, or quarantine honestly

- **Stabilize (preferred).** The diff removes the non-determinism and the test
  still asserts exactly what it asserted before. A "fix" that weakens the
  assertion, deletes the case, or wraps the test in a blanket retry is not one —
  a retried flaky test still lies, it just lies more slowly.
- **Quarantine (only when the cause needs evidence you do not have).** Propose
  the repo's own skip/quarantine mechanism **plus** a tracking note saying what
  is flaky and why it could not be settled from the source. Name it as the
  stopgap it is; a silent skip is how a suite quietly stops testing something.

## How to verify before you claim

- **Point at the line, not the file.** The sleep, the shared fixture, the real
  timer, the unseeded generator.
- **State the failing interleaving in one sentence** — "this awaits a 100ms
  sleep and then asserts the row exists; under CI load the insert has not
  committed yet."
- **Check the runner's configuration before blaming the test.** Parallel workers
  with a shared database, a global setup that runs once for tests that assume it
  runs each time, or a default timeout too tight for the slowest machine are
  suite-level causes with suite-level fixes.
