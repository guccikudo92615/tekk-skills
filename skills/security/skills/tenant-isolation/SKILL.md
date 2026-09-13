---
name: tenant-isolation
description: 'Audits whether one customer can reach another customer''s data — row-level security coverage and policy correctness, the service-role bypass, tenant scoping taken from session state rather than the request, and cross-tenant leaks through joins, caches, exports and admin paths. Use when reviewing migrations, database policies, tenant-scoped queries, or any shared-table schema.'
---

# Skill: tenant-isolation

**Skill:** the boundary between customers. One question, asked of every path that
touches shared tables: *if user A crafts this request, can they see or change
user B's rows?* Per-object permission inside one tenant is the auth-and-access
skill; you own the tenant line itself.

This is the finding class that ends companies, and it is never scale-gated: one
attacker, one leaked table.

## What to evaluate

1. **Policy coverage.** Every table holding customer data has row-level security
   *enabled*, and a policy that actually restricts. Read the migrations, not the
   dashboard: a table created in a later migration than the "enable RLS" sweep
   is the classic gap. `using (true)` for the authenticated role is coverage on
   paper and none in practice.
2. **Policy correctness.** The policy compares the row's owner to the *session's*
   identity (`auth.uid()`, a session GUC), never to a value the client can send.
   Check each command separately — `select` policies that are right while
   `insert`/`update` have `with check` missing let a user write rows they cannot
   read. Table owners bypass RLS unless it is forced.
3. **The service-role escape hatch.** A privileged key bypasses every policy by
   design. Find every place it is used: is it isolated to a server-side admin
   client, or is the same client used for ordinary user requests (making every
   policy decorative)? Is it reachable from the browser bundle or an edge
   function without its own authorization?
4. **Application-layer scoping.** Every query filters by the tenant from the
   *session*, not from the request body or a path parameter the caller chose.
   Grep the ORM calls for a tenant/workspace/org filter and find the ones
   missing it; check that shared helpers (`findById`, `getBySlug`) take a tenant
   argument rather than trusting the caller.
5. **The paths people forget.** Joins that reach an unscoped table through a
   scoped one; aggregate/count endpoints that leak existence; search indexes
   built across tenants; exports, CSV imports, webhooks, cron jobs and admin
   tooling that run without a tenant context; caches keyed without the tenant
   (one user warms it, the next reads it).
6. **Least privilege underneath.** The application role holds only the grants it
   needs. An application connecting as a superuser (or with blanket
   `grant all … to app_user`) turns any injection into a total compromise, and
   makes every policy above moot.

## How to verify before you claim

- **Prove the absence from the schema.** For "no policy on `X`", cite the
  migration that creates `X` and show no `enable row level security` /
  `create policy` for it anywhere; for a weak policy, quote the policy body.
- **Follow one query end to end.** Show the request field that reaches the
  filter, or the filter that is missing, at `file:line` — a table name in a
  schema file is not evidence that user data crosses.
- **Say which layer you are proposing.** Both layers matter (axiom 10): a
  database policy for the durable guarantee, an application-side scope for the
  paths that run as a privileged role. Name which one is missing rather than
  proposing "add tenant checks".

`references/rls-patterns.md` carries the concrete before→after shapes (enabling
and forcing RLS, per-command policies, session context, least-privilege grants)
— open it when the fix's exact form is the thing in question.
