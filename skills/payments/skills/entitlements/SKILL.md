---
name: entitlements
description: 'Audits what the customer can actually do after the money moves — whether every grant has a matching revoke, whether cancellation, expiry and failed renewals reach a defined state, and whether access is decided from real subscription state rather than a stale cache. Use when reviewing provisioning, plan/tier gating, credits, trials, or dunning logic.'
---

# Skill: entitlements

**Skill:** access, in both directions. Most billing code is written on the happy
path — someone pays, something is switched on — and the reverse path is written
later, or never. You own the reverse path and the states in between: cancelled,
expired, past-due, trialing, refunded, disputed.

The archetype finding: **cancelled-but-still-active.** Nobody complains, so it
is never discovered — the customer keeps premium and the founder keeps a
subscription line that produces nothing.

## What to evaluate

1. **Grant/revoke symmetry.** For every place access is granted, find the code
   that removes it. Walk the list of ways access should end: subscription
   deleted, period ended without renewal, trial expired, final dunning attempt
   failed, refund issued, dispute lost, seat removed from a team. A grant with
   no matching revoke is the finding; name the specific end-event that has no
   handler.
2. **The state machine, or the absence of one.** A failed renewal should move
   the account through defined states (grace → retry → suspend → revoke), not
   silently keep full access forever, and not hard-cut on the first failure
   (which churns a customer whose card simply expired). Check that the states
   are driven by the provider's real status and period end, not by an assumed
   duration or a local timestamp written at purchase.
3. **Where access is decided.** The gate reads server-authoritative state. Watch
   for a tier cached in a session, a JWT claim minted at login and never
   refreshed, a client-side flag, or a copy of the plan denormalised at signup
   and never updated — each keeps serving the old entitlement after a change,
   for as long as the cache lives.
4. **Downgrade and upgrade paths.** A downgrade that grants the lower tier but
   never removes the higher tier's resources (extra seats, projects over the new
   limit, stored data past a quota); an upgrade mid-cycle that double-charges or
   silently loses proration. Say what the code does today and what the customer
   experiences.
5. **Trials.** A trial that starts without an end, an end that nobody enforces,
   a trial-to-paid conversion that re-provisions from scratch (duplicating
   credits or resetting usage), or a second trial obtainable by the same
   customer.
6. **Credits, quotas and usage.** If entitlement is metered rather than binary:
   is the balance decremented atomically, does a failed payment stop the meter,
   is a refund reflected, and can the balance go negative or be double-granted
   by a retry? (The retry mechanics themselves are the webhooks skill; the
   *balance* semantics are yours.)

## How to verify before you claim

- **Grep both directions.** Find the grant call sites and the revoke call sites
  and compare the sets. Missing revoke code is provable by absence — say where
  it *would* live (the handler for the end event) and that nothing calls it.
- **Follow one cancellation end to end.** From the provider event through to the
  gate a user hits, naming each hop. If the gate reads a cached value, say how
  long the stale window is; that duration is the finding's blast radius.
- **Do not assert live drift.** "Customers are cancelled in Stripe but active
  here" needs the connector to confirm. Propose the missing revoke path from the
  code; flag reconciliation as the follow-up rather than claiming a number you
  cannot see.
