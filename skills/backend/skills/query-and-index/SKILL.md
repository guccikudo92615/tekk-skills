---
name: query-and-index
description: 'Audits how the app reads — N+1 queries, filters and joins on unindexed columns, unbounded or deep-paginated scans, over-fetching, and ORM calls that emit something pathological. Use when reviewing queries, repositories, ORM models, or any endpoint whose cost grows with the table.'
---

# Skill: query-and-index

**Skill:** the queries this change makes, and what they cost as the table grows.
The schema they hit is the schema-and-migrations skill; the handler's logic is
api-correctness's. You own "this read is wrong, unbounded, or will full-scan".

Enrichment note: the index and pagination material below draws on Supabase's
Postgres best-practice references (`supabase/agent-skills` →
`supabase-postgres-best-practices`), re-stated for a code-reading audit.
`references/index-choices.md` carries the concrete shapes.

## What to evaluate

1. **N+1.** A query issued per row — a query inside a loop/`map`/`Promise.all`,
   or an ORM relation lazily loaded per element. Propose the batched form (an
   `IN (…)`, a join, the ORM's eager-load) and **state the counts**: "1 + one
   per order → 2 queries". This is the finding most likely to be invisible in
   the diff and obvious in production.
2. **Missing indexes.** A column used in `WHERE`, a `JOIN` condition, or
   `ORDER BY` with no index behind it — read the migrations to confirm absence
   rather than assuming. Two specific cases worth checking every time:
   - **Foreign keys are not indexed automatically.** An unindexed FK makes both
     the join and `ON DELETE CASCADE` scan the child table.
   - **Column order in a composite index.** Equality columns first, range last;
     an index on `(created_at, status)` does not serve `status = ? AND
     created_at > ?` well, and two single-column indexes are not a substitute.
   Where the query reads few columns, a covering index avoids the table lookup;
   where it always filters to a subset (`WHERE deleted_at IS NULL`), a partial
   index is smaller and faster to maintain.
3. **Unbounded and pathological reads.** A list query with no `LIMIT`;
   `SELECT *` pulling wide or heavy columns nothing uses; deep `OFFSET`
   pagination that scans and discards (cursor/keyset pagination is the fix);
   an accidental cartesian join; a `count(*)` over a growing table on a hot
   path; loading rows to filter them in application code.
4. **ORM misuse.** Raw SQL built by string interpolation of a value (propose
   the parameterised form — note the injection overlap for Security, fix the
   query here); a call that silently emits a different query than it reads like;
   a relation loaded to use one field; a transaction wrapper used per row.
5. **Reads on the write path.** A handler that re-reads what it just wrote, a
   validation that queries per item, a serializer that resolves relations one at
   a time — each turns one request into many round trips.

## How to verify before you claim

- **Read the schema, not just the query.** For "unindexed", cite the migration
  or model definition and show the index is absent. A finding the schema
  contradicts is not a finding.
- **Point at the loop for an N+1.** The iteration and the per-iteration query,
  both at `file:line`.
- **Name the numbers you have, not ones you don't.** Query counts, the scan, the
  columns, the table's role. Do NOT invent milliseconds — measuring latency is
  the Performance loop's connector-fed job. "Full scan, unbounded, on a table
  that takes a row per signup" is the honest shape of this skill's evidence.
- **Where the engine's behaviour decides it, say which engine.** Index
  semantics, `NULL` handling and planner behaviour differ between Postgres,
  MySQL and SQLite.
