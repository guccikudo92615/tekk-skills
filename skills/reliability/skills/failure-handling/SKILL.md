---
name: failure-handling
description: 'Audits what the code does once something has already failed — empty catches, errors logged and then continued past on a write, unhandled promise rejections, unawaited writes, a failure that returns success, and multi-step work left half-applied with no compensation or recovery. Use for ordinary service and handler code with no dependency, queue or resource surface in sight.'
---

# Skill: failure-handling

**Skill:** the error path, read as if it will run — because it will. A caught
error that is not handled is worse than an uncaught one: the uncaught one
crashes loudly and gets fixed, while the caught one turns a failure into a
plausible-looking success and gets shipped.

This is the skill for ordinary code, and its findings are almost always sins of
omission. Read every `catch`, every error branch, every place a promise is
created, and ask what the CALLER now believes.

## What to evaluate

1. **Swallowed errors.** An empty `catch`, a `catch` that logs and continues
   past a write, a `.catch(() => {})`, an error branch that returns a default as
   though nothing happened. Ask what state the system is in after the swallow:
   if the answer is "half-applied, and nobody knows", that is a finding whatever
   the log line says.
2. **Unawaited and floating work.** A promise created and not awaited, a
   fire-and-forget write inside a handler that returns immediately, a
   `void doThing()` on a path whose success the response implies. The failure
   arrives with no context, after the request is gone — and in some runtimes it
   takes the process with it.
3. **Failures that report success.** A handler returning 200 with an error
   inside the body nobody checks, a function returning `null` for both "not
   found" and "the lookup failed", a boolean return that collapses three
   outcomes into two. Distinguishing "it did not happen" from "we do not know"
   is the whole job here.
4. **Partial application with no recovery.** Write A, call B, write C, and B
   fails: what is left behind? Either make it atomic, or make the leftover
   recoverable — a compensating action, a state the next run can resume from, or
   a record that something needs attention. Silence is the one option that is
   never right.
5. **Catching too broadly.** One `try` around thirty lines, so a typo, a
   validation failure and a provider outage are all handled identically — and
   the recovery that fits none of them is applied to all three. Propose the
   narrower boundary, or the discrimination inside it.
6. **Error context destroyed.** Re-throwing a new error without the cause,
   stringifying an exception into a message, catching and re-raising a generic
   type. The bug is not the failure, it is that the next person cannot tell what
   failed — a fix that keeps the cause chain is cheap and pays every incident.

## How to verify before you claim

- **Say what the caller believes.** The strongest form of this finding is a
  sentence about consequence: "the charge succeeded, the entitlement write threw,
  the handler returns 200, so the user paid and has no access."
- **Check whether something upstream handles it.** A framework error handler,
  middleware, or an outer boundary that logs and reports may already do the job.
  A local empty catch under a competent global handler is a different, smaller
  finding — name which one you found.
- **Check whether the error is genuinely ignorable.** Some are: a cache write
  that fails, best-effort telemetry, an optional enrichment. The test is whether
  the operation's promise to the caller still holds. If it does, say so and move
  on rather than proposing ceremony.
- **Propose the handling, not just the detection.** Log with context, surface to
  the caller, retry, compensate, or fail loudly — name which, and why that one
  fits this path.
