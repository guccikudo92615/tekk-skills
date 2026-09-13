---
name: uptime-and-heartbeats
description: 'Audits unattended work for a watcher — a deployed service with no health endpoint or a shallow always-green one, a health check nothing pings, a scheduled or background job with no check-in, and a check-in carrying no monitor config so no monitor exists to go late. Use when reviewing servers, cron, queues, workers, schedulers or CI workflow files.'
---

# Skill: uptime-and-heartbeats

**Skill:** the thing nobody is waiting for. A user notices a broken page in
seconds; nobody notices a nightly job that stopped running, or an API that has
been down since a deploy at 2am, until the consequences arrive. Your question is
always *how long would this be broken before anyone knew?*

Two halves of one job: the service says "I am alive", and something asks.

## What to evaluate

1. **A deployed, long-running service with no health endpoint.** An HTTP or SSR
   server, an API, a worker exposing a port. A static site, a library or a
   one-shot CLI has nothing to health-check — skip it rather than forcing one.
2. **A health check that always says yes.** A route returning 200
   unconditionally while the database is unreachable is worse than none: it
   converts an outage into a monitor that lies. Liveness ("the process is
   running") and readiness ("it can actually serve") are different questions;
   readiness must return a failing status when a critical dependency is down.
   Keep it cheap enough to hit every 30 seconds, reachable without an auth wall,
   and leaking no versions or secrets.
3. **A health endpoint with nothing watching it.** The endpoint alone is an
   emitter, not a monitor. Where the repo's CI can reach the deployment, propose
   the watcher as code — a scheduled workflow that fails on a bad response, with
   its URL in a repo variable rather than hardcoded — and be explicit about who
   the failure notifies and what one-time setup remains.
4. **Scheduled and background work with no check-in.** Crons, `setInterval`
   loops, queue consumers, nightly batches, scheduled functions. Each should
   report a successful completion, and long ones a start too.
5. **A check-in at the wrong boundary.** Reporting at the TOP of the job defeats
   the purpose — a run that dies halfway still checked in. The signal is
   completion.
6. **A check-in with no monitor config.** A bare check-in with no schedule and no
   grace period creates nothing that can be late, so nothing ever alerts. Where
   the project already runs a monitoring SDK, propose the check-in carrying its
   schedule, margin and max-runtime so the monitor upserts itself on first run.
   Where no SDK is present, propose a provider-agnostic heartbeat behind config
   that no-ops when unset — and say that wiring it to a monitor is the remaining
   manual step.

## How to verify before you claim

- **Establish that it is really deployed or really scheduled.** Read the
  deployment config, the process manifest, the scheduler registration. A worker
  that exists in the repo but runs nowhere is not a gap.
- **Check what already watches it.** A platform health probe in the deployment
  config, an existing scheduled workflow, a load balancer check, an uptime
  monitor referenced in the README — any of these may already cover the service.
- **Read the check body, not just its existence.** The finding is often that the
  endpoint exists and checks nothing.
- **Name the time-to-notice you are closing.** "This nightly reconciliation job
  reports nothing; if it stopped, the first symptom would be a customer noticing
  a wrong balance" is a finding. "No monitoring" is not.
