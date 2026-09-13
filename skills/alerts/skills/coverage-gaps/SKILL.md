---
name: coverage-gaps
description: 'Computes the critical paths that exist in the code minus the alert rules that exist in the project, and proposes the missing alert with a concrete condition and threshold — payment and billing failures, inbound webhook errors, auth failures, error-rate and latency breaches on hot endpoints. Use when auditing application code for failures nothing would report.'
---

# Skill: coverage-gaps

**Skill:** subtraction. Build two sets and take the difference.

1. **The critical paths that exist in this code** — the ones whose silent
   failure has real cost.
2. **The paths already covered** — from the GROUND TRUTH block in your brief,
   never from assumption.

What remains is the finding. One gap, sized and specified well enough that the
user can say yes without further thought.

## What counts as a critical path

- **Payments and billing** — checkout, charge, subscription lifecycle, dunning,
  and the provider's webhook handler. A failure here costs money directly and is
  usually invisible from the UI.
- **Inbound webhooks generally** — signature verification and event processing.
  A broken webhook drops events silently, and the symptom appears days later as
  missing state.
- **Auth** — login, token refresh, provisioning, session issuance. A failure
  locks users out of everything and generates support before telemetry.
- **The hot endpoints** — the highest-traffic routes, where an error-rate or
  latency change is a real incident rather than noise.
- **Unattended work** — a scheduled job or queue consumer whose failure nobody is
  waiting on.

A CRUD endpoint on a rarely-used settings page is not a critical path. Say what
makes the one you picked critical.

## How to propose the alert

- **Name the path and the failure**, at `file:line` — "payment charge failures on
  `POST /checkout` (`server/routes/checkout.ts:88`)".
- **The condition** — what the rule matches: a new issue on that path, an error
  with a specific message or tag, an error rate over a window, a latency
  percentile breach. Ground it in what the provider can actually match AND in
  how the code reports the failure. If the path swallows its error, the rule can
  never fire — say that the reporting has to come first.
- **A concrete threshold**, chosen from the path's importance and its likely
  volume. A threshold that would fire on a single stray error on a
  thousand-request path is how alert fatigue starts.
- **Routing and severity** — where it should go and how loud, noting that the
  user picks the channel.

## How to verify before you claim

- **Check the GROUND TRUTH block for an existing rule that already covers it,
  including one whose name does not match the path.** A rule that fires on any
  new issue in the project technically covers everything — say so, and make your
  finding the narrower rule that would be genuinely more useful, rather than
  claiming a gap that is not one.
- **Confirm the path exists and is reachable.** A route behind a feature flag
  that is off, or a handler with no caller, is not worth an alert.
- **Confirm the failure would be reported.** Trace the error path: does it throw,
  is it captured, does anything reach the provider? An alert on an unreported
  failure is a rule that will sit green through an outage.
- **Prefer one alert on the money path over three on adjacent ones.** Every rule
  you add is a future interruption; propose the one you would defend at 3am.
