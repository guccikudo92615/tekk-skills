---
name: tracing-and-metrics
description: 'Audits flight-critical paths for whether their success, failure and latency are observable — payments, auth, outbound third-party calls, queue work and hot handlers that emit no spans or metrics, spans that record no error status, and instrumentation that carries unbounded or sensitive attributes. Use for ordinary service and handler code, or when reviewing tracing, telemetry or metrics setup.'
---

# Skill: tracing-and-metrics

**Skill:** the question you could not answer. Pick a path that matters — a
checkout, a login, a call to the payment provider, a queue job — and ask what
you could say about it right now: how often it succeeds, how often it fails, how
long it takes, and whether that changed after the last deploy. Where the answer
is "nothing", the path is unobservable, and the first you will know of a
degradation is a customer describing it.

Crash reporting tells you it broke. This tells you it is slow, failing
intermittently, or quietly getting worse.

## What to evaluate

1. **Flight-critical paths with no instrumentation.** Payments, checkout and
   billing; auth and session; every outbound third-party call; background and
   queue work; the hottest request handlers. Rank by blast radius — a money or
   auth path outranks a cold CRUD handler, always.
2. **Partial instrumentation, which reads as covered.** Instrumented means all
   three of success, failure and latency are visible. A `try/catch` that
   rethrows is not instrumentation. A counter of attempts with no outcome
   dimension cannot tell you a failure rate. A span that never records its error
   status makes a failing path look like a fast one.
3. **The outbound call in particular.** When a third party degrades, the only
   defence is having measured it: duration, status and error recorded at YOUR
   boundary, attributed to the provider. This is the highest-value single span in
   most applications and the one most often missing.
4. **Instrumentation that changes behaviour.** A wrapper that swallows the
   exception it recorded, alters a return value, or adds an await that changes
   ordering. Observation must be inert — if the proposal changes control flow, it
   is out of scope.
5. **Attributes that will hurt.** Unbounded ids, raw user input, emails, tokens —
   high-cardinality attributes explode a metrics backend's cost and sensitive
   ones are a leak in a new place. Propose the identifying dimensions the team
   can actually filter on: route, workspace, provider, outcome.
6. **Naming that cannot be aggregated.** A span or metric name carrying an id or
   a formatted string produces one series per request. Names are for grouping;
   the specifics are attributes.

## How to verify before you claim

- **Look one layer up first.** Framework auto-instrumentation, a tracing
  middleware, a decorator on the base class, an SDK that already wraps HTTP
  clients — any of these may already cover the path, and proposing a duplicate
  span both misleads and costs. Name what you checked.
- **Match the stack exactly.** If the repo uses OpenTelemetry, use its helper and
  its naming conventions; if it has a metrics client or a structured logger with
  a duration field, extend that. Never introduce a second telemetry library.
- **State the question the instrumentation answers.** "There is no way to see
  the provider's error rate, so an outage looks like a slow checkout" beats "add
  tracing".
- **Check the cost side of the proposal.** Sampling, cardinality, and where the
  data lands are part of the finding, not an afterthought — instrumentation is
  one of the easier ways to build a surprising bill.
