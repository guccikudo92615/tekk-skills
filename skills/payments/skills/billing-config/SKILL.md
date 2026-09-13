---
name: billing-config
description: 'Audits the configuration that decides what gets sold — price and plan ids split between env and code, test-mode vs live-mode keys, plan-to-feature maps that drift from the provider, currency/tax assumptions, and webhook endpoint settings. Use when reviewing pricing config, plan constants, env handling, or a price/plan change.'
---

# Skill: billing-config

**Skill:** the lookup tables between the product and the provider. Nothing here
is clever code — it is ids, keys, modes and maps — which is exactly why it rots:
a price is renamed in the dashboard, one of the three places that referenced it
is updated, and checkout starts 400ing or silently sells the wrong tier.

## What to evaluate

1. **One source of truth for price/plan ids.** Count the places a given price id
   appears: an env var, a constant, a seed file, a test fixture, a doc. Two
   copies is drift waiting to happen; the fix is one map, read everywhere.
   *(The `p60 → p960` rename is this codebase's own instance of this class.)*
2. **Config that fails loudly.** A missing or unknown price id should stop the
   deploy or the request with a clear error — not fall through to `undefined`, a
   default tier, or a silently skipped grant. Look for the fallback branch that
   turns a config mistake into a wrong entitlement instead of an error.
3. **Test vs live mode.** Keys, price ids and webhook secrets are mode-scoped and
   cross-wiring them is a whole class of bug: a live key with test price ids, a
   test webhook secret verifying live events, seed data carrying the wrong mode.
   Check how mode is selected and whether anything asserts that the key, the
   ids and the endpoint agree.
4. **The plan→feature map.** The table that says what each plan grants (limits,
   seats, features). Does it include every plan the provider actually sells,
   including grandfathered and legacy ones? What happens for a plan id it does
   not know — deny, or grant the default? Say which, and which is right here.
5. **Amounts, currency and tax assumptions.** Amounts in the provider's minor
   units (cents) versus a float in the app; a currency assumed to be one value;
   tax or VAT handled by the provider but re-derived locally. Any place the app
   recomputes an amount the provider already computed is a divergence risk —
   prefer reading the provider's number.
6. **Endpoint and secret hygiene as config.** The webhook endpoint's configured
   event list versus the events the code handles (an event the code handles but
   the endpoint never sends is dead code; the reverse is a silent gap), and a
   signing secret that is per-endpoint rather than shared.

## How to verify before you claim

- **Show both copies.** A drift finding needs the two locations and the values
  that differ (or the one that is now missing upstream). One reference is not
  drift.
- **Prefer the fix that removes the second copy** over a fix that syncs it —
  synchronisation is the bug that keeps recurring.
- **Say what breaks and when.** "Checkout 400s for the Pro plan today" and "the
  next price change silently sells the wrong tier" are different severities;
  name which one you have, from the code.
