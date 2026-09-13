---
name: api-correctness
description: 'Audits the trust boundary of a request handler — input validated where it enters, errors that fail loudly instead of reporting success, and the defaults that decide behaviour under load (pagination caps, timeouts, body limits, rate limits, CORS). Use when reviewing routes, controllers, API handlers or middleware.'
---

# Skill: api-correctness

**Skill:** what the handler accepts and what it does when things go wrong.
Legit-but-malformed input, a failure reported as success, a default that is fine
at ten users and hostile at ten thousand. Retries and concurrency are the
idempotency skill; the query the handler runs is query-and-index's.

## What to evaluate

1. **Validation at the boundary, once.** The handler reads `req.body`/`query`/
   `params` (or a job reads its payload) and passes it into business logic, a
   write, or an outbound call without validating shape, type and range. The fix
   is one gate at the entry using the repo's existing validator (zod, joi,
   class-validator, a framework pipe) — not defensive checks scattered
   downstream, which is how two layers end up disagreeing about what is valid.
   Watch for validation that runs only on the client, and for a schema that
   accepts extra keys straight into a spread-into-database write.
2. **Error handling that hides or corrupts.** An empty `catch`, a
   catch-and-continue on a write path, a missing `await` so a rejection escapes
   the handler, an operation that partially succeeds and returns 200, an error
   swallowed into a default value that then gets persisted. Propose handling
   that fails loudly and unwinds the partial work.
3. **What the client is told.** A raw stack or internal string returned to the
   caller; a 200 with an error body that clients treat as success; a 500 where
   the real answer is 400 (so the client retries something that can never
   succeed); an error shape that differs per route so no client can handle it
   generically.
4. **Unsafe or missing defaults.** A list endpoint with no pagination cap (an
   unbounded query that degrades as the table grows), no timeout on an outbound
   call (one slow dependency ties up the request), no body-size limit on an
   upload, no rate limit on an expensive or abuse-prone route, permissive CORS
   with credentials. Each is one line to fix and expensive to discover in
   production.
5. **Contract drift.** A response shape changed without a version or a
   deprecation path; a field made required that older clients don't send; an
   enum value added that clients switch on exhaustively. Say who breaks.
6. **Auth-adjacent correctness that is NOT the exploit.** A route that needs the
   session's user but reads an id from the body, so an honest client with a
   stale cache acts on the wrong record. (When the same line lets an *attacker*
   do it deliberately, that is Security's — name the seam, propose once.)

## How to verify before you claim

- **Trace the path, don't pattern-match.** Untrusted input → does it actually
  reach a state-changing sink unguarded? A global validation pipe, an ORM that
  parameterises, a framework's body limit, or middleware already applied to the
  router are all reasons there is no finding. Read a sibling handler first —
  that is the app's real pattern.
- **Name the request that breaks it.** "POST this body and the handler stores
  `undefined` as the amount" is a finding; "input is not validated" is a
  category.
- **Check the failure path is reachable.** An error branch nothing can trigger
  is defence-in-depth, not `high`.
