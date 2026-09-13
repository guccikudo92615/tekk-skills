---
name: schema-and-migrations
description: 'Audits the shape of the data and how it changes — migrations that must survive a non-atomic deploy, constraints that keep bad rows out, data types, and connection/pool configuration. Use when reviewing migration files, schema definitions, ORM models, or database client setup.'
---

# Skill: schema-and-migrations

**Skill:** the durable layer. A bad query is slow; a bad migration is an outage,
and a missing constraint is corruption that accumulates silently until someone
has to write a cleanup script. Query shape is the query-and-index skill; you own
the schema it runs against and the change that gets it there.

## What to evaluate

1. **Deploy-window safety — the load-bearing rule.** A deploy is not atomic:
   for a window, the NEW schema runs under OLD code, and/or new code runs under
   the old schema. Both must work. So in a single migration, flag:
   - `DROP` or `RENAME` of a table or column,
   - a column type change,
   - `NOT NULL` added without a default to an existing table,
   - a tightened constraint or a narrowed enum.
   Each breaks one side of the window. Propose the **expand-contract** rewrite:
   additive now (`ADD COLUMN` nullable or defaulted, new table, new index),
   backfill, ship code that stops using the old shape, drop it in a LATER
   migration. Prefer `IF NOT EXISTS` so a re-run is a no-op.
2. **Locks taken by the migration itself.** A migration that rewrites a table or
   takes an exclusive lock blocks writes for its duration — on a big table that
   is the outage, even though the DDL is "additive". Adding a column with a
   volatile default, creating an index without `CONCURRENTLY`, and validating a
   constraint in the same statement that adds it are the usual causes; the safe
   forms are `ADD CONSTRAINT … NOT VALID` followed by a separate `VALIDATE`, and
   `CREATE INDEX CONCURRENTLY` outside a transaction.
3. **Idempotent, reversible migration code.** `ADD CONSTRAINT IF NOT EXISTS` is
   not valid SQL in Postgres — a guard needs a `DO $$ … pg_constraint … $$`
   block or an equivalent check. A migration that fails halfway and cannot be
   re-run is worse than one that never ran.
4. **Constraints that keep bad rows out.** A foreign key that should exist
   (orphans accumulate), a column that should be `UNIQUE` (duplicate emails,
   slugs, or the idempotency key another skill is about to rely on), `NOT NULL`
   where the code already assumes it, a `CHECK` on a range or enum, an
   `ON DELETE` behaviour that orphans or cascades further than intended. When
   adding one to a table with existing rows, say what the backfill/cleanup step
   is — the migration will fail on dirty data otherwise.
5. **Data types.** A money amount in a float; a timestamp without a timezone; a
   too-narrow integer for an id that will keep counting; text where an enum or a
   FK belongs; JSON used as a bag of fields that queries then filter on (and
   would need a GIN index to do adequately). These are cheap to fix early and
   expensive later — say which it is here.
6. **Connections and pooling.** A client created per request instead of a shared
   pool; an unbounded pool that can exhaust the server's connection limit under
   load; a connection acquired and not released on an error path; a transaction
   held open across an external call. One non-obvious trap: under
   transaction-mode pooling, connections are shared between statements, so
   **named prepared statements break** — most drivers use the unnamed form, but
   a hand-rolled `PREPARE` or a session-scoped setting will fail intermittently
   and look like a flake.

## How to verify before you claim

- **Read the migration and the current schema together.** "Missing constraint"
  needs the table definition; "unsafe migration" needs the statement quoted.
- **Say which side of the deploy window breaks, and how.** "Old code still
  writes `name`, this migration drops it, so every write 500s until the deploy
  finishes" is the finding. Naming the rule alone is not.
- **Check the repo's tooling and conventions first** (its ORM's migration
  runner, whether migrations run automatically or by hand, whether the project
  documents its own rule). Propose in that idiom — a fix that does not run the
  way this project runs migrations is not a fix.
