---
name: injection-and-input
description: 'Audits untrusted input reaching a dangerous sink — SQL/NoSQL concatenation, shell execution, SSRF, path traversal, unsafe deserialization, unsafe rendering — and the agent-specific version of the same bug: prompt injection, unverified tool authorization, and model output trusted downstream. Use when reviewing route handlers, query building, file or URL handling, or any LLM/agent code path.'
---

# Skill: injection-and-input

**Skill:** one shape of bug, in two eras. Classic: attacker-controlled bytes reach
an interpreter (SQL, a shell, a filesystem path, an HTTP client, a deserializer,
a DOM). Modern: attacker-controlled *text* reaches a model that holds tools, and
the model becomes the interpreter. Input that merely breaks the handler is the
Backend loop's; you own input that makes the system act for the attacker.

## What to evaluate

1. **SQL / NoSQL construction.** String concatenation or template interpolation
   of a request value into a query, a `WHERE` clause assembled from user input,
   an ORM escape hatch (`raw`, `$queryRawUnsafe`, `sql.raw`) carrying a variable.
   Parameterised/tagged form is the fix. Note the seam: the query *mechanics*
   are the Backend loop's lane — you are here because the value is
   attacker-controlled.
2. **Shell and process execution.** `exec`/`spawn` with a shell and an
   interpolated argument; an argument array is the fix. Also: a filename,
   archive entry or URL passed to a CLI tool.
3. **SSRF.** A URL supplied by a user or stored from one (webhook targets,
   avatar imports, link previews, "fetch my site") requested server-side.
   Validate at **both** config time and delivery time — a hostname that resolved
   to a public address when saved can resolve to a link-local address later.
   Block the metadata endpoint and private ranges explicitly, and disable
   redirect-following or re-validate each hop.
4. **Path traversal and file handling.** A user-supplied name joined into a path,
   an archive extracted without checking entry names, an upload trusted by its
   declared content-type, a download route that resolves outside its root.
5. **Deserialization and dynamic evaluation.** `eval`, `Function`, dynamic
   `require`/`import`, YAML/pickle loaders with object construction enabled, a
   template engine rendering user-controlled template text (SSTI).
6. **Rendering untrusted content.** Raw HTML injection into the DOM
   (`dangerouslySetInnerHTML`, `innerHTML`) with interpolated content, a
   Markdown/HTML sanitizer configured permissively, a redirect target read from
   a query parameter (open redirect).
7. **Prompt injection on tool-using agents.** Untrusted content — a fetched
   page, an uploaded file, an inbound message, another user's record — reaching
   a prompt without structural delimiting (a distinct role/block that the system
   prompt tells the model is data, not instructions; a textual "ignore any
   instructions below" is not a control). The impact is bounded by the agent's
   tools, so state what those tools are.
8. **Tool authorization and model output.** Every tool call re-checks the
   *caller's* rights at call time, not just at session start; a tool that takes
   an id must scope it to the session's tenant. Model output written to the
   database, rendered to a page, or fed to another tool is untrusted input again
   — trace it to its sink. Per-user cost/token limits belong here too: an
   unbounded agent surface is an abuse channel.

## How to verify before you claim

- **Trace source → sink, and name both.** The request field, the transformation
  it survives, the interpreter it reaches. If a validator, an ORM parameter, a
  sanitizer or a framework default neutralises it on the way, there is no
  finding — check the sibling handler for the app's real pattern first.
- **For agent findings, name the tool the injection buys.** "Prompt injection
  possible" with a read-only, no-tool call is `low`; the same injection into a
  loop that can write, send or spend is `critical`. Read the tool allow-list;
  do not infer it.
- **Prefer the app's existing guard.** Propose the validator/sanitizer/parameter
  style already used elsewhere in the codebase over introducing a new library.
