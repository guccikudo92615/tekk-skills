---
name: crash-reporting
description: 'Audits every application entrypoint for error reporting — a server, worker, serverless function, CLI or client runtime that initializes no error-tracking SDK, an init that misses unhandled rejections, and an SDK configured without environment, release or sane sampling so its output cannot be acted on. Use when reviewing entrypoints, bootstrap and server startup code, or error-tracking setup.'
---

# Skill: crash-reporting

**Skill:** where does an unhandled error GO? Follow each entrypoint of this
application to the moment something throws and nobody catches it, and ask who
finds out. In an uninstrumented process the answer is a line in a log nobody
reads, or nothing at all — the process restarts and the evidence is gone.

An application usually has more entrypoints than the team remembers: the API
server, a background worker, an edge or serverless function, a CLI, the browser
runtime. Instrumenting one and forgetting a second deployed process is the
commonest shape of this gap.

## What to evaluate

1. **An entrypoint with no SDK init at all.** Unhandled exceptions and rejections
   there vanish. Propose the initialization the framework expects, at the TRUE
   entrypoint — before the app boots, so an error during startup is reported too,
   which is exactly when errors are most likely.
2. **An init that misses a class of failure.** Unhandled promise rejections
   uncaught, worker threads or child processes outside the handler's scope, a
   framework error boundary that swallows before the SDK sees it, an error
   middleware registered before the routes it should wrap. Ordering is
   load-bearing here and easy to get wrong.
3. **Reports that cannot be acted on.** A bare `init()` with no environment, so
   staging noise buries production; no release or version, so nothing can be
   attributed to a deploy; a sample rate that silently drops most events; no
   user or workspace context, so an error cannot be traced to who hit it.
4. **The client runtime.** A frontend crash that leaves no trace is the one class
   of failure the server-side SDK will never see, and often the one users
   actually experience.
5. **Reporting that leaks.** Request bodies, headers, tokens or PII attached to
   an event by default. Propose the scrubbing config alongside the init rather
   than as a follow-up — the leak ships with the instrumentation.

## How to verify before you claim

- **Enumerate the entrypoints from the deployment config**, not from memory:
  process manifests, service definitions, function handlers, package scripts.
  Then check each one for an init. Say which you enumerated.
- **Trace where the init actually runs.** An import in a module that only loads
  after the app has booted, or behind a flag that is off in production, is an
  init in name only.
- **Check for a wrapper.** Some platforms and frameworks auto-instrument, and
  some repos centralize the init in a shared bootstrap module the entrypoint
  imports. Proposing a duplicate init is a real failure mode — it double-reports
  and doubles the bill.
- **Name what would be lost.** "The queue worker runs in its own process with no
  init, so a crash mid-job leaves only the restart in the platform log" is a
  finding.
