---
name: harness
description: 'Audits the agent loop itself — whether the ''agent'' is a real harness with tool use, state and error recovery, and whether its prompts, tools and untrusted-input handling hold up in production. Use when reviewing agent code, tool definitions, MCP servers, model calls, or prompt-injection surface.'
---

# Skill: harness

**Skill:** the agent loop itself — is the "agent" a real harness, and do its
prompts, tools, and input handling hold up under real use? (Corpus/RAG
machinery is the retrieval skill's lane, not yours.) Classify
each dimension below as production-grade / partial / absent. **Ground every
verdict in the artifact itself**: before calling a dimension production-grade,
have **code-explorer** read the specific artifact (the loop body, the prompt
text, the tool schema, the error path) and cite `file:line`. Production-grade
asserted from the *shape* of the system, without reading the specific call, is
how a real gap gets rubber-stamped as fine. If you can't cite what you
inspected, you haven't inspected it — classify it as unassessed, not solid.

## What to evaluate

1. **The loop.** Is the "agent" a single stateless completion call, or a real
   harness — tool use, multi-turn state, error recovery? Single-call-
   pretending-to-be-an-agent is the highest-leverage finding on early
   projects. Check the error path specifically: a retry that re-sends the same
   failing input unchanged is a placebo, not recovery.
2. **Prompts.** System prompts versioned and structured, or string soup inline
   in handlers? Can two call sites drift apart because the prompt is
   copy-pasted rather than shared?
3. **Tools.** A coherent tool surface with typed schemas, or ad-hoc JSON
   parsing of free-text model output? Are tool errors fed back to the model in
   a form it can act on, or swallowed?
4. **Input safety.** Untrusted content flowing into prompts unguarded;
   injection surface on tool-using agents (fetched pages, user uploads, inbound
   messages that can steer the loop); PII discipline in what gets sent to the
   provider.

## How to triage

The loop file's sizing rules apply. Within this lane specifically: a broken
error path on the *core* agent outranks a missing nicety on a side flow, and a
prompt-injection hole on a tool-using agent is never scale-gated — one crafted
input is enough.
