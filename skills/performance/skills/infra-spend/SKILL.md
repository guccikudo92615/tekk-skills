---
name: infra-spend
description: 'Audits the machine bill — a cron or poll where an event would do, a schedule far tighter than its inputs change, an always-on service that could be on-demand, work redone every tick with no cache, egress from over-fetch or missing cache headers, chatty calls to a metered API, and paid services still wired but unused. Use when reviewing cron, jobs, queues, workers, deployment or infrastructure config.'
---

# Skill: infra-spend

**Skill:** the invoice for work the product did not need to do. Every scheduled
job, worker, service and outbound byte in this diff, asked: *what does this cost
every day, and what would be lost if it ran less often, later, or not at all?*
The finding is always the same shape — the same outcome, less machine.

Recurrence is what makes this skill worth having. A wasteful thing on a request
path is a latency bug; a wasteful thing on a schedule is a subscription.

## What to evaluate

1. **Polls that should be events.** A cron that fetches to check whether
   something changed, where the source can push — a webhook, a queue message, a
   database notification. The poll burns a run every interval to usually find
   nothing, and its cost scales with the interval, not with the work.
2. **Schedules tighter than their inputs.** A job running every minute over data
   that changes daily, a nightly rebuild of something that changes weekly, a
   sync whose upstream publishes hourly. Propose the frequency the inputs
   justify, and say what determines it.
3. **Always-on for intermittent work.** A worker, container or service kept
   running for work that arrives in bursts, where on-demand or scale-to-zero
   does the same job. Also its inverse-in-disguise: a "warm" instance kept alive
   purely to avoid a cold start nobody measured.
4. **Work redone every tick.** A stable result recomputed each run — a full
   re-index where an incremental pass would do, a re-download of an unchanged
   artifact, a report regenerated identically. Propose the cache or the
   watermark, with what invalidates it.
5. **Egress and payload.** Wide selects and over-fetch moving bytes nobody
   reads; assets and cacheable responses served without cache headers, so every
   client re-fetches from the origin; a chatty per-row call to an API priced per
   call, where a bulk endpoint or a local cache collapses the count.
6. **Dead weight.** A SaaS SDK still imported and configured for a feature that
   was removed, two services doing one job (two analytics tools, two error
   trackers, an overlapping queue), a provisioned integration that is connected,
   billed, and never called. Prove it: show the import has no call sites, or
   that the second tool's events are a subset of the first's.

## How to verify before you claim

- **Code or config is the evidence — utilisation is not available to you.** You
  cannot see CPU headroom, instance sizing, storage tiers or committed-use
  discounts, so do not speculate about them. Propose only what the repo proves:
  a schedule's frequency, a poll's shape, a service with no callers, a missing
  cache header, an unbounded payload. Where the real answer needs utilisation
  data, say so and stop.
- **Read the schedule AND what it does.** A cron running every minute that exits
  immediately on a cheap check is not the same finding as one that queries on
  every tick. Open the job body before sizing it.
- **Quantify by recurrence, not by guess.** "Runs 1,440×/day and issues one
  query each time to find nothing" is a sized claim. A dollar figure derived
  from an assumed traffic volume is not — give the per-tick shape and the
  frequency, and let the reader multiply.
- **Check for the consumer before calling anything dead.** Dynamic imports,
  config-driven wiring, a workflow file, an environment variable read at boot —
  a grep for the symbol is necessary but not sufficient. Say what you searched.
- **Same outcome is the constraint.** A cheaper schedule that makes data staler
  is a product decision, not a saving: state the staleness the change
  introduces, and if it matters, propose the event-driven form instead.
