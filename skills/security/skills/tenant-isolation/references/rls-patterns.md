---
title: Row-level security patterns — the shapes a correct fix takes
impact: CRITICAL
tags: rls, multi-tenant, policies, least-privilege
source: adapted from supabase/agent-skills → supabase-postgres-best-practices (security-rls-basics, security-privileges)
---

Open this when the *form* of the fix is the question. The skill (whether there is
a hole at all) is in `../SKILL.md`.

## 1. Application filtering is not isolation

**Incorrect — the only guard is the query the developer remembered to write:**

```sql
select * from orders where user_id = $current_user_id;
-- one forgotten filter, one injection, one admin client → every row
select * from orders;
```

**Correct — the database refuses, whatever the query says:**

```sql
alter table orders enable row level security;

create policy orders_user_policy on orders
  for all
  to authenticated
  using (user_id = auth.uid());

-- owners and superusers bypass RLS unless it is FORCED
alter table orders force row level security;
```

`enable` without a policy denies everything (safe but usually a bug — the app
breaks loudly). A policy without `enable` is inert (unsafe and silent): check
for both.

## 2. Per-command policies — `using` reads, `with check` writes

`for all … using (…)` covers reads and the *existing* row on a write, but not
the row being written. A user can then insert or update rows into another
tenant that they cannot read back.

```sql
create policy orders_select on orders for select to authenticated
  using (user_id = auth.uid());

create policy orders_insert on orders for insert to authenticated
  with check (user_id = auth.uid());          -- the NEW row must be theirs

create policy orders_update on orders for update to authenticated
  using (user_id = auth.uid())                 -- may touch only their rows
  with check (user_id = auth.uid());           -- …and may not reassign ownership
```

## 3. Identity comes from the session, never the payload

```sql
-- Incorrect: the client tells the database who it is
create policy p on orders for all using (user_id = current_setting('request.user_id')::bigint);
-- when request.user_id is set from a header or body field, the policy is a formality

-- Correct: a value the request cannot forge
create policy p on orders for all to authenticated using (user_id = auth.uid());
```

Server-side code that sets a session variable per request (`set local
app.current_user_id = …`) is fine **only** when that value is derived from a
verified session, and `set local` (transaction-scoped) is what keeps it from
leaking across pooled connections.

## 4. Least privilege under the policies

```sql
-- Incorrect: any injection becomes total compromise
grant all privileges on all tables in schema public to app_user;

-- Correct: the app role holds what the app needs and no more
create role app_writer nologin;
grant usage on schema public to app_writer;
grant select, insert, update on public.orders to app_writer;
grant usage on sequence orders_id_seq to app_writer;
-- no delete, no ddl, no access to tables it never reads
```

## 5. The service-role key

A privileged key bypasses every policy above. Two rules worth proposing when
they are absent: it is instantiated once, server-side, in an admin client that
ordinary request handlers do not import; and every path that uses it performs
its own authorization, because the database will not.

Reference: <https://supabase.com/docs/guides/database/postgres/row-level-security>
