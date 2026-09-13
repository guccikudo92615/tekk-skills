---
name: money-and-webhooks
description: 'Audits whether someone can get paid features without paying, and whether inbound provider webhooks can be forged, replayed or pointed at another account. Use when reviewing checkout, subscription and entitlement code, or any webhook receiver that grants access or credits.'
---

# Skill: money-and-webhooks

**Skill:** the attacker who wants free service. Every path where money decides
access — checkout, the webhook that confirms it, the state it writes — read as
"what can I send to get the grant without the payment?" Whether the billing record is
*correct* (a retry double-charging an honest customer, a revoke that never runs)
is the Payments loop's job; you own the deliberate bypass.

## What to evaluate

1. **Client-supplied money values.** A price, amount, plan id, seat count,
   trial length, coupon or currency read from the request body and used to
   create the checkout or grant the entitlement. Server-authoritative catalogue
   lookup is the fix; the finding is the line where the client value flows in.
2. **Webhook signature verification — on the RAW body.** The receiver verifies
   with the provider's constructor against the unparsed body. Two common
   breaks: a global JSON body-parser mounted before the webhook route (the
   signature can no longer be computed, so verification is skipped or wrapped in
   a permissive try/catch), and a verification failure that logs and continues.
3. **Replay and idempotency.** The same signed event delivered twice must grant
   once. Look for a unique constraint on the provider event id (a check-then-act
   in application code is not enough under concurrency), and a timestamp
   tolerance so an old captured event cannot be re-sent.
4. **The identity chain.** Trace `event → customer → user/tenant`: is the account
   resolved from a value the *buyer* controlled at checkout (`client_reference_id`,
   metadata, an email string), and is it re-verified against the stored customer
   id? A metadata field the attacker chose, trusted at fulfilment, is
   subscription hijacking.
5. **Checkout races.** Two concurrent completions for one purchase, a redeem
   endpoint with no lock, a credit grant that reads-then-writes. Propose the
   locked transaction or the unique constraint that makes the second attempt
   fail.
6. **Enforcement after the grant.** A revoked, expired or downgraded
   subscription that still passes the gate: a cached tier that never
   invalidates, a grace period computed from a client date, an unknown provider
   status treated as active, a "self-healing" reconciliation that re-grants what
   was revoked (axiom 4).
7. **The non-2xx retry storm.** A handler that returns 500 on an event it
   actually processed makes the provider redeliver forever — a self-inflicted
   amplification that also multiplies every idempotency weakness above.

## How to verify before you claim

- **Name the request.** The finding is "POST this body to this route and you get
  X for free", grounded at the line where the value is trusted. A missing check
  that no reachable path exercises is defence-in-depth, not `critical`.
- **Read the provider's real behaviour, not your memory of it.** Which events
  this integration receives, what the constructor verifies, whether the account
  identifier is attacker-settable — check the code and the provider's current
  docs before sizing.
- **Cross-check the neighbour lane once.** If the same line is also a Payments
  finding (a correctness bug rather than a bypass), say which framing you are
  proposing and why — never file both.
