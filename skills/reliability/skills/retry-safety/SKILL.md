---
name: retry-safety
description: 'Audits state changes that can run more than once — a webhook redelivered, a job re-run after a crash, a queue that is at-least-once, a user double-submitting — for the dedupe that makes the second execution harmless: idempotency keys, unique constraints, atomic upserts, and side effects that cannot be replayed. Use when reviewing webhooks, queues, jobs, cron, or any retried write.'
---

# Skill: retry-safety

**Skill:** the same operation, twice. For every state change on a path that can
be retried or redelivered, ask: *what does the second execution do?* The
dangerous answer is not an error — it is success. A charge that succeeds twice
looks fine in the logs and wrong on the statement.

Your job starts one step earlier than the write: **trace the retry source**.
Something has to make the second execution possible — a provider that redelivers
on non-2xx, an at-least-once queue, a client retry, a cron with no overlap
guard, a resumed job. Name it, then follow it to the side effect.

## What to evaluate

1. **State changes with no dedupe on a retryable path.** A create, charge,
   provision, enqueue, email or outbound call with no idempotency key and no
   unique constraint. The fix is durable, not conditional: a unique index the
   second write collides with, an idempotency key stored WITH the result so the
   replay returns the original, or an atomic upsert. An application-level "have
   we done this already?" read followed by a write is not idempotency — two
   concurrent replays both pass the read.
2. **The provider's actual redelivery contract.** Payment and messaging
   providers redeliver on any non-2xx, sometimes for days. So a handler that
   throws on an event it has already processed does not just log noise — it
   requests the retry storm that multiplies every other weakness here. Check
   what status the code returns on the duplicate path.
3. **Out-of-order and late delivery.** Redelivery is not ordered. An event
   processed after a newer one can overwrite fresher state with staler state.
   Look for handlers that write blind, and propose the guard: a version, a
   timestamp comparison, or a state machine that refuses backwards transitions.
4. **Crash points inside a multi-step operation.** Write, then call the
   provider, then write again — with the second execution starting from the top.
   The question is not "will it be retried" but "which of these steps runs
   twice". Propose the boundary that makes the whole thing resumable: a
   transaction where it is one datastore, a recorded step marker where it is not.
5. **Jobs and cron that overlap themselves.** A slow run and its successor
   working the same rows, a poll with no visibility timeout so a second worker
   takes work still in flight, a "process pending" query with no claim step. The
   fix is a lock, a lease, or `SELECT … FOR UPDATE SKIP LOCKED`.
6. **Side effects that cannot be taken back.** Some second executions cannot be
   made harmless — the email is sent, the webhook has fired. Those need the
   dedupe BEFORE the effect, not compensation after it.

## How to verify before you claim

- **Name the retry source explicitly.** "Stripe retries this webhook on any
  non-2xx, and the handler inserts unconditionally" is a finding. "Not
  idempotent" alone is not.
- **Read the schema before proposing a key.** A unique constraint may already
  make the write idempotent; conversely an `idempotency_key` column with no
  unique index enforces nothing at all, which is the more common shape.
- **Check the infrastructure's guarantees.** Some brokers dedupe, some drivers
  retry with backoff by default, some frameworks wrap the handler in a
  transaction. If the platform already provides it, there is no finding.
- **Say what the duplicate costs.** A duplicated audit row and a duplicated
  charge are not the same severity. Money, provisioning and outbound
  communication rank first.
