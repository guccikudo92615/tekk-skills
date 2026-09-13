---
name: auth-and-access
description: 'Audits who is allowed to do what — route-level ownership and role checks, IDOR, TOCTOU, entitlement enforcement — and the signup/login/reset/session flows that hand out identity in the first place. Use when reviewing auth middleware, session handling, permission checks, account-lifecycle flows, or any route that reads someone else''s object by id.'
---

# Skill: auth-and-access

**Skill:** the *access decision* — who may perform this operation on this object,
and whether the flows that establish identity can be subverted. Tenant-wide data
boundaries are the tenant-isolation skill; getting paid features without paying
is money-and-webhooks. You own "this caller should not have been allowed, and
here is the request that proves it".

## What to evaluate

1. **Missing or wrong authorization on a route.** An endpoint that authenticates
   but never checks *ownership* — `GET /orders/:id` returning any order to any
   logged-in user (IDOR). Read the handler to the data access: is the tenant/user
   scope applied in the query, or only assumed? Also check the inverse
   asymmetry: a `GET` that is guarded and the sibling `DELETE`/`PATCH` that is
   not.
2. **Role checks that are the wrong role, or absent on the mutation.** Billing
   and member-management endpoints requiring `admin` where the product means
   `owner`; a UI that hides an action while the API still serves it; an
   authorization check that runs on the read path and not on the write path.
3. **TOCTOU — permission checked outside the transaction.** The check reads
   state, then the mutation acts on state that may have changed (a revoked seat,
   a downgraded plan, a transferred object). Propose moving the check inside the
   `FOR UPDATE`/advisory-locked transaction that performs the write.
4. **Deny-by-default, or allow-by-omission?** Establish how a NEW route gets
   protected in this codebase: a global middleware with an explicit public
   allowlist (safe), or a per-route opt-in guard (a route added next week will
   be public by accident). If it is opt-in, that structural finding usually
   outranks any single unguarded route — name the routes currently unguarded as
   the evidence.
5. **Account lifecycle — the shadow half of auth.** The reset/verification/
   session flows themselves:
   - a password-reset token that is reusable, non-expiring, or not invalidated
     after use or after a password change;
   - a session not rotated on login or on privilege change (fixation);
   - login, reset and OTP endpoints with no rate limit or lockout (credential
     stuffing, reset-bombing, OTP brute force);
   - an email-change or verification flow an attacker can hijack for takeover;
   - OAuth/SSO missing `state` or PKCE (CSRF on the callback).
6. **Entitlement enforcement.** Premium operations gated on server-authoritative
   tier, not a client claim; grace periods computed from the real
   `currentPeriodEnd`; a cache that keeps serving the old tier after a
   downgrade; an unknown provider status silently treated as "active".
7. **Comparison and response hygiene.** Secret/token comparisons that are not
   constant-time; auth responses that distinguish "no such user" from "wrong
   password" in body, status or timing (axiom 5 in practice).

## How to verify before you claim

- **Walk one concrete request.** Name the method, path, and the field an
  attacker controls; then show the line where the object is loaded and the
  absence of the scope. "The handler doesn't validate" is not a finding until
  you have followed it to the data.
- **Check the middleware chain first.** A global guard, a framework policy, or a
  scoped ORM default may already enforce what looks missing in the handler —
  read a sibling route to learn the app's real pattern before you call a route
  unguarded.
- **Rank by what the request gets you.** Reading another user's object outranks
  a missing role check on an idempotent internal endpoint; a takeover path
  (reset/email-change) outranks both.
