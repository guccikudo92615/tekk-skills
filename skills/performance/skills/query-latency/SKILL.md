---
name: query-latency
description: 'Audits how the app reads data under load — a query inside a loop (N+1), a per-row lookup that should be one batched or joined query, unbounded or unindexed queries on a hot path, SELECT * pulling heavy columns, and pagination that scans the whole table. Use when reviewing database access, ORM calls, repositories, or any handler that reads before it responds.'
---

# Skill: query-latency

**Skill:** the read path as the table grows. Every query in this diff, asked:
*how many times does this run per request, and how much does the database have
to touch to answer it?* A query that is instant on a thousand rows and fatal on
a million is the archetype — it passes review, passes tests, and degrades on a
Tuesday.

## What to evaluate

1. **N+1.** A database or API call inside a loop, or a per-row lookup that
   should be one batched query or one join. Count it out loud: "one query for
   the list, then one per item, so a 200-item page issues 201 queries." The fix
   is the batched form — an `IN` clause, a join, a dataloader-style batch — and
   the count is the evidence.
2. **Unbounded reads.** A query with no `LIMIT`, an endpoint that returns every
   row, a scan the app filters in memory, a "load all then count" where the
   database could aggregate. These are the ones that fail suddenly rather than
   gradually: fine until one workspace has 50,000 rows.
3. **Unindexed filters and joins on a hot path.** A `WHERE` or `ORDER BY` on a
   column with no index, a join on an unindexed foreign key, a `LIKE '%…'` that
   cannot use one. Check the schema and the existing indexes before claiming —
   a composite index may already cover it, and column ORDER in a composite index
   decides whether the query can use it.
4. **Over-fetch.** `SELECT *` or an ORM default pulling wide columns — blobs,
   JSON documents, long text — that the caller never reads. The cost is both
   database work and bytes on the wire; propose the column projection.
5. **Pagination that scans.** `OFFSET` deep into a large table re-reads every
   skipped row. Keyset/cursor pagination on an indexed column is the fix, and
   the deeper the page the bigger the win.
6. **Queries in the wrong place.** A query per render, a query inside a
   serializer, a lookup in a template — each looks harmless locally and
   multiplies by whatever calls it.

## How to verify before you claim

- **Check whether the path is already failing, not just slow.** `sentry_issues`
  lists what is throwing in production right now. A handler that is both slow and
  erroring is a different, more urgent proposal than one that is merely slow, and
  the error usually names the cause.
- **If Sentry is mounted, take the measurement.** Your run context lists the
  connector tools you hold. A transaction name plus a p95 is a lead; the span/op
  breakdown is the finding. Ask which operation inside the request owns the time
  before you name the query — proposing an index for a route whose 2.4s is
  actually one external HTTP call with no deadline is a confident, wrong,
  expensive proposal. And query the path the diff touched even when the poll was
  quiet: the poll raises what has CHANGED, so a route that is steadily the
  slowest thing you have — and was looked at once — is absent from it by design.

- **Read the schema and the existing indexes.** The single most common false
  positive in this skill is proposing an index that already exists under another
  name or as the prefix of a composite one. Migration files are what was
  intended; if the Supabase tools are mounted, the live schema is what is true.
- **Establish the loop count.** Trace what actually calls the query and how many
  times per request. "Inside a loop" is only a finding when the loop has real
  cardinality — say what bounds n, and if n is bounded and tiny, drop it.
- **Check what the ORM emits, not what the code reads like.** Lazy relations,
  eager-loading defaults and hidden joins mean the SQL is often not the shape the
  call site suggests.
- **Name what stays identical.** Same rows, same order, same values — a batched
  rewrite that changes ordering or drops a tie-break is a behaviour change.
