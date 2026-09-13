---
name: event-coverage
description: 'Audits the product''s real user journeys for whether anything captures them — acquisition and signup, the activation moment where a user first gets value, the core repeated action the product exists for, and conversion or revenue steps — and proposes the specific capture calls and where they go. Use when reviewing onboarding, signup, checkout, pricing or any flow a user moves through.'
---

# Skill: event-coverage

**Skill:** the journey, walked end to end, asking at each step *would we know a
user got here?* Not whether an analytics tool is installed — whether the
specific moments that decide whether this business works are recorded.

The four that matter, in the order they matter:

1. **Acquisition and signup** — landing, sign-up started, account created. Where
   users come from and how many finish.
2. **Activation** — the moment a user first gets real value. This one is
   product-specific and it is the one most often missing: first project created,
   first message sent, first integration connected, first report generated.
   Naming it correctly is half the work, because everything downstream is
   measured against it.
3. **Core action** — the repeated thing the product exists for. The heartbeat
   that separates a user who came back from one who logged in.
4. **Conversion and revenue** — checkout started, plan selected, upgraded,
   subscribed, cancelled. Including the failure branches, which is where the
   money question actually lives.

## What to evaluate

- **A step in one of those four flows with no capture call at all.** Trace the
  real code path a user takes and name where the call belongs, at `file:line`.
- **A capture that fires at the wrong moment.** On the button click rather than
  the successful result, so failed attempts count as conversions; on render
  rather than on the action; after a redirect that half the users never
  complete. Fire on the outcome, not the intent — and where both are useful,
  say so explicitly ("started" and "completed" are two events, and their ratio
  is the finding).
- **A flow instrumented on one surface only.** The web signup is captured and
  the invite-link signup is not, so a whole cohort is invisible and the numbers
  quietly disagree with the database.
- **Server-side actions captured only in the client.** Anything that can succeed
  after the tab closes — a webhook-driven upgrade, an async provisioning step —
  needs to be captured where it actually happens.
- **Events with no way to join them.** A capture with no stable user or account
  identifier cannot be assembled into a funnel at all; anonymous events before
  signup need the alias step that connects them to the account afterwards.

## How to verify before you claim

- **Read the flow, do not infer it from route names.** The activation moment in
  particular is not guessable from the file tree — find where the product
  actually delivers its first value.
- **Search for existing captures before proposing one.** They may be wrapped in a
  helper, fired from a hook, or emitted server-side from an event handler. Name
  what you searched for.
- **Propose placement, not intent.** "Capture `checkout_completed` in
  `server/routes/webhook.ts:142`, where the subscription is confirmed, with
  plan and amount as properties" is a proposal. "Track conversions" is not.
- **Do not instrument everything.** Every event proposed is one someone has to
  maintain and pay for. Four good events on the spine beat forty on the buttons.
