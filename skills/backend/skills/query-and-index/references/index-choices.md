---
title: Index and pagination shapes — what a correct fix looks like
impact: HIGH
tags: indexes, composite, covering, partial, foreign-keys, pagination, explain
source: adapted from supabase/agent-skills → supabase-postgres-best-practices (query-*, schema-foreign-key-indexes, data-pagination, monitor-explain-analyze)
---

Open this when the *form* of the index or pagination fix is the question. Whether
there is a problem at all is `../SKILL.md`.

Examples are Postgres; the shapes carry to other engines, the syntax does not.

## 1. Foreign keys are not indexed for you

```sql
-- Incorrect: the FK constraint exists, the index does not
create table orders (
  id bigint generated always as identity primary key,
  customer_id bigint references customers(id) on delete cascade
);
select * from orders where customer_id = 123;  -- Seq Scan
delete from customers where id = 123;          -- scans orders to cascade

-- Correct
create index orders_customer_id_idx on orders (customer_id);
```

The cascade case is the one people miss: deleting a parent scans the child table
per row, so a "slow delete" report is often this.

## 2. Composite column order: equality first, range last

```sql
-- Incorrect: two single-column indexes, so the planner bitmap-ands them
create index orders_status_idx on orders (status);
create index orders_created_idx on orders (created_at);

-- Correct for `where status = ? and created_at > ?`
create index orders_status_created_idx on orders (status, created_at);
```

An index on `(created_at, status)` serves that query badly: once a range is used,
columns to its right cannot be used for filtering. The same index also serves
`where status = ?` alone (leftmost prefix), which is why one composite often
replaces two singles.

## 3. Partial and covering variants

```sql
-- Partial: the query always filters to a subset, so index only that subset
create index orders_active_idx on orders (customer_id) where deleted_at is null;

-- Covering: the query reads few columns, so keep them in the index and skip
-- the table lookup entirely
create index orders_lookup_idx on orders (customer_id) include (status, total);
```

Partial indexes are smaller and cheaper to maintain; covering indexes trade
write cost and size for read speed. Both are worth proposing only when the
query shape genuinely matches.

## 4. Pagination: cursor, not deep OFFSET

```sql
-- Incorrect: page 500 scans and throws away 10,000 rows
select * from orders order by created_at desc limit 20 offset 10000;

-- Correct: keyset — carry the last row's sort key
select * from orders
 where (created_at, id) < ($1, $2)
 order by created_at desc, id desc
 limit 20;
```

Add `id` as the tiebreaker so rows with equal timestamps cannot be skipped or
repeated between pages.

## 5. Proving it

The claim "this uses an index" is checkable: `EXPLAIN ANALYZE` shows `Index
Scan` vs `Seq Scan`, the row estimate vs the actual, and where the time went. A
loop run cannot execute SQL, so cite the schema and the query shape — and where
the fix is worth a measurement, say that `EXPLAIN ANALYZE` on the real table is
the confirmation step, rather than asserting a speedup you did not measure.
