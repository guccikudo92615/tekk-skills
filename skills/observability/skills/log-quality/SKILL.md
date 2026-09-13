---
name: log-quality
description: 'Audits what the code leaves behind when it fails — an error branch or empty catch with no record, unstructured messages with no context or correlation id, log levels that make real failures invisible in noise, and PII, tokens or secrets written into a log line. Use when reviewing logging setup, loggers, redaction helpers, or any failure path.'
---

# Skill: log-quality

**Skill:** the incident, three weeks from now, reconstructed from what was
written down. Read each failure path and ask: *is there a record, does it say
which request and which user, and could you follow it without the author in the
room?* Then ask the opposite question, which matters just as much: *is anything
in here that should never have been written down?*

## What to evaluate

1. **Failures with no record.** An empty `catch`, a `catch` that neither logs nor
   rethrows, an error branch that returns a default silently. The proposal is
   the log at the right level with the context to act on — and note the swallow
   itself, because a failure that returns success is a bug in its own right even
   once it is logged.
2. **Records with no context.** `console.error(err)` alone tells you something
   broke somewhere. The useful record carries what identifies the work: the
   request or correlation id, the workspace or user, the operation, the inputs
   that matter — enough that the line can be found and joined to others from the
   same request.
3. **Unstructured where structure is needed.** Free-text string interpolation in
   a service that already ships a structured logger means those lines cannot be
   filtered, aggregated or alerted on. Match the logger the codebase already
   uses; do not introduce a logging framework it does not have.
4. **Levels that hide the signal.** Everything at `info` (so nothing stands out),
   real failures at `warn` (so nothing pages), debug logging left on in
   production (so volume buries the line that mattered — and costs money to
   store).
5. **Sensitive data in log lines.** Emails, names, tokens, API keys, session
   cookies, full request bodies, whole database rows spread into a message. Also
   the quieter forms: an error object that carries the query with its parameters,
   a third-party response logged whole, a stack trace with credentials in a URL.
   Propose the redaction where the logger is configured — a serializer or
   scrubber that applies everywhere — rather than fixing one call site and
   leaving the pattern.
6. **Logging that costs more than it gives.** A log line inside a hot loop or per
   row of a batch, at a volume nobody will ever read. Sampling or aggregation is
   the fix, and it belongs in the same proposal as the line you are adding.

## How to verify before you claim

- **Check the logger's configuration before proposing redaction at a call
  site.** Many setups already redact by key. If they do, the finding is either
  gone or narrower: a field the scrubber does not know about.
- **Check for an upstream handler.** A framework error handler or middleware may
  already log every failure with context — in which case a local addition is
  duplication, and the honest finding is smaller.
- **Say what you would have wanted during an incident.** "This catch returns
  `null` on a provider failure with no record, so a checkout that silently does
  nothing leaves no trace to correlate with the user's complaint" is a finding.
- **Do not propose logging as a substitute for handling.** If the right answer is
  to fail loudly, say so — a well-formatted record of an error being ignored is
  still an error being ignored.
