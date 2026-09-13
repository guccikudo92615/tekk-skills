---
name: idempotency-and-jobs
description: 'Audits anything that can happen twice or interleave — non-idempotent state changes under retry or redelivery, check-then-act races, multi-step writes with no transaction, webhook receivers, and queue/cron consumers without retry, dead-letter or lock discipline. Use when reviewing webhooks, jobs, queues, cron, or any handler that creates, charges or provisions.'
---

# Skill: idempotency-and-jobs

**Skill:** the second execution. Every state change in this diff, asked twice:
*what happens if this runs again, or runs concurrently with itself?* In
production it will — clients retry, providers redeliver, queues are
at-least-once, users double-click, and two requests arrive in the same
millisecond.

## What to evaluate

1. **Idempotency of state-changing operations.** A POST/PUT, webhook or job that
   creates, charges, provisions or emails with **no idempotency key and no
   dedupe**. Propose the durable form: a unique constraint the write can lean
   on, an idempotency key stored with the result, or an upsert
   (`INSERT … ON CONFLICT DO UPDATE`) that collapses the second run. An
   application-level "already done?" read followed by a write is NOT idempotent
   — two concurrent runs both pass the read.
2. **Check-then-act races.** Read seat count → insert; read balance →
   decrement; find-or-create. Under concurrency both callers see the old value.
   The fix is a database-level guarantee: a unique constraint, `SELECT … FOR
   UPDATE` inside the transaction, an atomic `UPDATE … WHERE balance >= x`, or
   an advisory lock keyed on the entity for work that spans tables.
3. **Multi-step writes with no transaction.** Create order + decrement
   inventory + write audit row, with a crash point between any two. Propose the
   transaction boundary — and check its shape: a transaction held open across
   an external HTTP call holds locks for the length of someone else's outage.
   Keep the external call outside, or make the work resumable.
4. **Deadlock and lock discipline.** Two code paths that lock the same rows in
   opposite orders will deadlock under load. Consistent lock ordering (always
   parent before child, always ascending id) is the fix, and it is invisible
   until it isn't. Same family: a long transaction that blocks a hot row, and a
   queue poll that should use `SKIP LOCKED` so workers don't serialise on the
   same head row.
5. **Webhook receivers.** Signature verified on the RAW body; every event type
   the provider actually sends either handled or explicitly ignored; a 2xx
   returned only for events durably taken, and a non-2xx only when redelivery is
   genuinely wanted — a 500 on an event already processed is a retry storm that
   multiplies every weakness above. Delivery is unordered: a handler that writes
   state blind can regress it when an older event lands late.
6. **Queues, jobs and cron.** No retry policy (a transient failure silently
   loses the work); no dead-letter, so one poison message wedges the consumer or
   vanishes; a job that swallows its own failure and looks done; a cron with no
   overlap guard, so a slow run and the next run process the same rows; work
   scheduled with no visibility timeout that another worker can pick up mid-
   flight.
7. **Compensation.** When a multi-step operation cannot be one transaction
   (it crosses services), is there a compensating action for the half that
   succeeded, or does the system just keep the inconsistency?

## How to verify before you claim

- **Name the concrete second execution.** "Stripe retries this webhook on any
  non-2xx and the handler inserts unconditionally" is a finding. "Not
  idempotent" alone is not — say what redelivers, and what the duplicate costs.
- **Check the schema before proposing a key.** A unique constraint may already
  exist and make the write idempotent; conversely an idempotency column with no
  unique index enforces nothing.
- **Check the queue's own guarantees.** Some brokers dedupe, some drivers retry
  with backoff by default, some ORMs wrap the callback in a transaction. If the
  infrastructure already provides it, there is no finding.
