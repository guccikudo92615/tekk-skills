---
name: dependency-boundary
description: 'Audits every call that leaves the process — HTTP, database, cache, queue, another service — for deadlines, retry policy, backoff and blast-radius containment: calls that can hang forever, tight retry loops that amplify an outage, no jitter, no cap, and a fragile dependency with nothing between it and the rest of the system. Use when reviewing API clients, SDK wrappers, connectors or integration code.'
---

# Skill: dependency-boundary

**Skill:** the moment your code depends on something it does not control. Every
outbound call in this diff, asked: *what happens to this process when the other
side is slow, flapping, or gone?* The answer should be a bounded, deliberate
degradation. Usually the answer is "it waits forever, and so does everything
behind it".

## What to evaluate

1. **Missing deadlines.** An HTTP request, database query, cache read, queue
   publish or internal service call with no timeout. Without one, the other
   side's hang becomes your hang, and the worker, connection or event-loop slot
   is held until something else gives up. Check the CLIENT's default before
   claiming — many SDKs ship an infinite default, some ship a sane one, and the
   difference decides whether there is a finding.
2. **Timeouts that are not budgets.** A 30s timeout on a call inside a request
   whose own budget is 5s protects nothing. Deadlines should shrink as you go
   deeper, and a retry must fit inside the remaining budget rather than
   multiplying it — three attempts at 10s each inside a 5s handler is a 30s
   worst case for a caller who left long ago.
3. **No retry where the failure is transient.** A network blip, a 429, a 503, a
   deadlock or a connection reset drops real work when the code treats it as
   final. Retry the *retryable* — and only the retryable: retrying a 400 or a
   validation failure just spends the same money for the same answer.
4. **Retries that amplify.** A `while` loop with no backoff, no jitter and no
   cap turns one slow dependency into a self-inflicted storm — and when many
   clients retry in lockstep, into a thundering herd at exactly the wrong
   moment. Propose exponential backoff with jitter and a bounded attempt count,
   and say what happens after the last attempt.
5. **Nothing containing the blast radius.** One fragile dependency reached from
   everywhere, with every request piling onto it while it is down, until the
   whole service is unavailable for features that never needed it. The remedies
   are a circuit breaker (stop calling for a while, fail fast) and a bulkhead
   (bound how much of the pool any one dependency can consume). Propose the
   simpler one that fits the codebase.
6. **No fallback for a non-essential dependency.** An optional enrichment, a
   recommendation service, an avatar host: when it fails the feature should
   degrade, not the page. Name what the degraded state should be.

## How to verify before you claim

- **Read the client's configuration, not the call site alone.** A wrapper,
  interceptor, base client or framework default often already sets the timeout
  and the retry policy. Say which file you checked; "no timeout on this line" is
  a false positive if the client sets one globally.
- **Establish that the failure is transient before proposing a retry**, and that
  the operation is safe to repeat. If it is not, the finding belongs to
  `retry-safety` — a retry added to a non-idempotent write is a new bug, not a
  fix.
- **Name the failure mode concretely.** "This provider call has no timeout, so a
  provider stall holds the request thread until the load balancer's 60s cut" is
  a finding. "Missing timeout" is a lint.
- **Say what happens after the last attempt.** Every retry proposal needs an
  answer: surface the error, queue for later, fall back, or fail the request
  cleanly. Retries that end in silence are a swallowed error with extra steps.
