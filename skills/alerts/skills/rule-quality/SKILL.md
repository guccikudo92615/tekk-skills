---
name: rule-quality
description: 'Audits alert rules that already exist for whether they would actually help — thresholds too tight to survive normal traffic or too loose to ever fire, rules watching a path whose failure is never reported, duplicate and overlapping rules, alerts with no owner or route, and coverage that drifted after the code moved. Use when reviewing alerting, monitoring or notification configuration.'
---

# Skill: rule-quality

**Skill:** the alert as it will be experienced. Take each rule in the GROUND
TRUTH block and ask two questions: *would this fire when it should?* and *would
it stay quiet when it should?* A rule failing either is worse than no rule,
because it is counted as coverage.

This is the run where the answer to "are we alerted?" is yes and the answer to
"would we find out?" is still no.

## What to evaluate

1. **Thresholds that cannot fire.** A rule requiring more failures than the path
   receives requests, a window longer than the outage would last, a condition
   matching an error string the code no longer emits. The tell is a rule that has
   never triggered on a path that has certainly failed.
2. **Thresholds that always fire.** A rule that goes off on background noise —
   any error at all on a path with a known steady rate of user-error, a latency
   threshold below the path's normal p95. These get muted, and a muted channel
   covers nothing. Propose the threshold that separates an incident from a
   Tuesday, using what the code and the traffic shape suggest.
3. **A rule watching a path that reports nothing.** The rule matches errors from
   a handler that catches and returns a default. Nothing is ever reported, so
   nothing ever matches. This is the highest-value finding in the skill, and the
   fix is upstream: the failure has to be reported before any rule can see it.
4. **Drifted coverage.** The rule references a route, tag, environment or
   release format that the code has since renamed. Alerting config rots exactly
   like a test does, and nothing turns red when it does.
5. **Duplicates and overlaps.** Several rules firing on the same failure into the
   same channel, so one incident arrives as five notifications. Consolidation is
   a real finding: it makes the remaining alert more likely to be read.
6. **Alerts with nowhere to go.** A rule routed to a channel nobody watches, or
   with no owner, or with no indication of what the recipient should do. An alert
   that does not imply an action is a notification.

## How to verify before you claim

- **Work only from the GROUND TRUTH block plus the code.** You cannot see firing
  history, mute state, or who read what. Say which of your claims would need that
  history to confirm, and frame them as the prediction they are.
- **Establish the path's normal behaviour before calling a threshold wrong.**
  Read what the handler does on ordinary bad input — a route that legitimately
  returns errors for user mistakes needs a different threshold from one where any
  error is a bug.
- **Trace the reporting before trusting a rule.** For any rule you are judging,
  check that the code path it watches genuinely surfaces its failures.
- **Do not propose a rule edit as an `## Execute` block.** The executor creates
  new-issue rules; it does not modify existing ones. A quality finding is a plain
  proposal describing the change the user makes in the provider.
