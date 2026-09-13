---
name: webhooks-and-fulfillment
description: 'Audits whether the payment provider''s events are all handled and whether fulfilment runs exactly once — missing money-critical events, at-least-once delivery with no dedupe, out-of-order updates, and the trial path that provisions nothing. Use when reviewing a payment webhook handler, checkout completion, or any code that provisions after money moves.'
---

# Skill: webhooks-and-fulfillment

**Skill:** the hand-off from provider to product. The provider emits an event;
your code must hear the ones that matter, act on each exactly once, and end up
in the state the provider is actually in. Whether the *entitlement* that results
is later revoked is the entitlements skill; you own everything up to and
including the write.

## What to evaluate

1. **Event completeness.** Read the handler's switch/router and compare it to
   the events this integration actually receives. Beyond the happy
   `checkout.session.completed`:
   - `invoice.payment_failed` / `invoice.payment_action_required` — an
     unhandled failed renewal leaves a lapsed customer fully provisioned.
   - `customer.subscription.deleted` / `.updated` — a cancel or downgrade that
     nothing handles means access never changes.
   - `charge.dispute.created` and refund events — a chargeback with no handler
     leaves the account looking paid.
   - **The trial trap:** `checkout.session.completed` fires for a card-required
     trial with `payment_status: 'no_payment_required'`. Fulfilment that only
     provisions when `payment_status === 'paid'` provisions **nothing** for
     every trial signup. *(This exact bug has shipped in production more than once.)*
   Name the event and what breaks without it, not "handle more events".
2. **Exactly-once fulfilment.** Providers deliver at least once and retry on any
   non-2xx, so the same event WILL arrive twice. Look for a durable dedupe: a
   unique constraint on the provider event id, an idempotency key, or a
   conditional write — not an `if (alreadyFulfilled)` read-then-write, which two
   concurrent deliveries both pass. The proposal is the constraint, not a flag.
3. **Ordering.** Delivery order is not guaranteed, and a retry of an old event
   can land after a newer one. A handler that writes state unconditionally can
   therefore *regress* it — re-provisioning a subscription that was cancelled a
   second earlier. Look for an ordering guard: compare the event's timestamp or
   the object's version against what is stored, or re-read the current object
   from the provider before writing. Flag handlers that write blind.
4. **Acknowledgement semantics.** A 2xx means "I have durably taken this"; a
   non-2xx means "send it again". Two failure shapes: work done *after* the
   response (or in a fire-and-forget promise) that can be lost, and a handler
   that returns 500 for an event it processed fine — an unhandled event type, a
   parse error on an unrelated field — which turns into an infinite retry storm
   against every weakness above.
5. **Verification on the raw body.** Signature verification needs the unparsed
   body; a JSON body-parser mounted ahead of the route breaks it. The *exploit*
   is Security's lane, but a webhook that cannot be verified — and is therefore
   skipped, or wrapped in a permissive catch — mis-fulfils for honest customers
   too, which is yours.
6. **What fulfilment actually does.** Follow it to the end: the plan write, the
   credit grant, the receipt email, the analytics call. A multi-step fulfilment
   with no transaction can half-apply (credits granted, plan not set) and the
   retry then double-applies the half that succeeded.

## How to verify before you claim

- **Read the handler's real event list** against the provider's current event
  reference — not a remembered list. If the integration is not Stripe, use that
  provider's names and semantics.
- **For a dedupe claim, show the write.** Point at the insert/update and the
  absence of a unique constraint or idempotency key in the schema. An
  application-level "already done?" check is evidence *for* the finding, not
  against it.
- **Check the SDK first.** Some client libraries verify, retry or dedupe for
  you; if the library already covers it, there is no finding.
