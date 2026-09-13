---
name: resource-limits
description: 'Audits what runs out under load — connection-pool starvation, a connection or transaction held across an await, unbounded queries and in-memory accumulators, streams and uploads with no size limit or backpressure, and listeners, timers or handles that leak. Use when reviewing pooling, connection or transaction handling, streaming, uploads, or any code that accumulates.'
---

# Skill: resource-limits

**Skill:** the finite thing. Connections, memory, file handles, sockets, worker
slots — each has a ceiling, and the code in front of you either respects it or
discovers it in production. These are the bugs that pass every test, because a
test never runs a hundred of anything at once.

The tell is almost always **unbounded**: a query with no limit, a buffer that
grows with input, a pool with no queue timeout, a loop that holds something
while it waits.

## What to evaluate

1. **Connection-pool starvation.** A connection or transaction checked out and
   held across an `await` on something else — an HTTP call, another query, a
   lock. Under concurrency every connection ends up waiting on someone else's
   latency, and the pool empties while the database sits idle. Also: acquiring
   without releasing on the error path, and a pool sized for one process running
   in many.
2. **Long or nested transactions.** A transaction open across an external call,
   or wrapping work that did not need to be transactional, holds locks for the
   duration of someone else's outage. Keep the external call outside the
   boundary, or make the work resumable.
3. **Unbounded reads into memory.** Load-all-then-filter, a full export built as
   one array, an accumulator that grows with rows rather than with results. Each
   is fine at seed-data scale and fatal at customer scale. Propose the bound: a
   limit, a cursor, streaming, or chunked processing — and say which one the
   caller can actually consume.
4. **Streams and uploads with no ceiling.** A body parser with no size limit, a
   file read into memory before it is written, an unbounded multipart upload, a
   download proxied through the process. Also missing backpressure: a producer
   faster than its consumer with an unbounded queue between them is a memory
   leak with a schedule.
5. **Unbounded concurrency.** `Promise.all` over a list of unknown length firing
   every request at once — against your own database, or against a third party
   that will rate-limit you. Propose a bounded worker count or a batching helper
   the codebase already has.
6. **Leaks.** Event listeners added per request or per render and never removed,
   timers never cleared, subscriptions never closed, caches with no eviction and
   no TTL — an in-memory map keyed by user id is a leak wearing a cache's
   clothes.

## How to verify before you claim

- **Establish the growth term.** Say what n is and what makes it grow: rows per
  workspace, items per request, concurrent users. If n is bounded by something
  small and structural, there is no finding.
- **Check the framework's defaults first.** Body-size limits, pool sizes,
  statement timeouts and idle-connection reaping are often configured a layer up
  — in middleware, in the pool constructor, in the deployment config. Name what
  you checked.
- **Prove the hold, not the shape.** For a pool claim, point at the line that
  acquires and the `await` that happens before the release. "This helper does I/O
  inside the transaction callback" is the evidence.
- **Say what the ceiling is and what happens at it.** "Pool of 10; each request
  holds a connection for the duration of the provider call, so 10 concurrent
  checkouts stall every other query" is a sized finding. "Could exhaust
  connections" is not.
