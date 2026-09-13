---
name: hot-path-blocking
description: 'Audits what a request waits on — synchronous I/O and heavy CPU in a handler, an external call with no timeout, work that should be queued or backgrounded, a response built without a cache, and payloads or assets that make the client wait. Use when reviewing routes, handlers, controllers, middleware, streaming or upload paths.'
---

# Skill: hot-path-blocking

**Skill:** the critical path of a single request. Follow it end to end and ask at
each step: *is the user waiting for this, and do they have to be?* Latency is
almost never one slow thing; it is a handler doing four things in sequence where
two of them could have happened later, in parallel, or not at all.

## What to evaluate

1. **Synchronous I/O on the request path.** A blocking file read, a synchronous
   hash or crypto round, an image transform, a large JSON parse or serialization
   done inline. In a single-threaded runtime this blocks every other request too,
   which is why one slow endpoint can flatten a whole service.
2. **External calls the user waits for.** A third-party API, a webhook fan-out,
   an email or notification send inside the handler. Ask whether the response
   truly depends on the result; if it does not, it belongs on a queue. If it
   does, it needs a **deadline** — an unbounded external call means your p99 is
   somebody else's outage.
3. **Sequential awaits that could be concurrent.** Independent fetches awaited
   one after another, where the latency is the sum instead of the maximum.
4. **Work redone every request.** A config parsed, a client constructed, a
   template compiled, a large structure rebuilt per call, where hoisting it to
   module scope or a memoized accessor costs nothing.
5. **A missing cache on a hot, stable read.** A response computed from data that
   changes rarely, requested constantly. Propose the cache WITH its invalidation
   story — a cache with no answer for "when is this wrong?" is a correctness bug
   waiting to be filed as a performance win.
6. **Payload and asset weight.** A response carrying far more than the client
   renders, an unbounded list embedded in a page, an uncompressed or
   unoptimized asset served from the origin on every load.

## How to verify before you claim

- **Establish that the path is hot.** A route hit once a day doing sync I/O is
  not a finding. Name what calls it and roughly how often — a request handler,
  a loop tick, a render path.
- **Check the framework's defaults first.** Compression, keep-alive, connection
  pooling, a static-asset cache header, a built-in timeout: half of what looks
  missing is configured a layer up. Say where you looked.
- **Say what the wait costs, in the units you have.** "Three sequential calls at
  ~200ms each → ~600ms of the handler's ~700ms" is a sized claim. Where you have
  a measured p95 from the runtime evidence, use it and drop the estimate.
- **Backgrounding changes semantics — say so.** Moving work off the request path
  means the caller no longer learns whether it succeeded. Name the failure
  handling (retry, dead-letter, a status the client can poll) as part of the
  proposal.
