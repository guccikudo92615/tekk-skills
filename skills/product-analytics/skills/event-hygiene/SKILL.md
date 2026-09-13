---
name: event-hygiene
description: 'Audits whether captured analytics can actually be used — is an SDK really initialized and firing, are names consistent and machine-parseable, do events carry the properties a question needs, is PII or a token being sent as a property, and is there a tracking plan or just an accumulated pile. Use when reviewing analytics setup, tracking helpers, or event definitions.'
---

# Skill: event-hygiene

**Skill:** the analyst six months from now — usually the same founder — trying to
answer a question from what was captured. The events exist. Can they be joined,
filtered, trusted, and named in a sentence? A pile of `button_clicked` with no
properties is nearly as blind as nothing, and it costs money to store.

## What to evaluate

1. **An SDK that is not actually running.** Initialization behind a key that is
   unset in production, inside a component that never mounts, after the events it
   should have captured, or blocked by a consent gate that never resolves. The
   code reads as instrumented and captures nothing — check the init path, not
   just its presence.
2. **Names that cannot be grouped.** `signup`, `Signed Up`, `user-signup` and
   `signupCompleted` for the same moment; names carrying an id or a formatted
   string, so every event is its own series. Propose the convention the repo can
   follow — object-action, one case, one tense — and apply it consistently rather
   than renaming one call.
3. **Properties that answer nothing.** An event with no dimensions cannot be
   segmented, so it can tell you a total and nothing else. The useful properties
   are the ones a question needs: plan, source, surface, variant, outcome, error
   reason. Equally, properties nobody will ever filter on are cost.
4. **PII and secrets as properties.** Raw emails, names, tokens, API keys,
   free-text user input, whole objects spread into the payload. This is the
   finding to lead with when you see it: it is a privacy exposure in a third
   party's system, and it is usually accidental. Propose the identifier plus the
   safe dimensions instead.
5. **Capture that ignores consent.** Where the product has a consent gate,
   tracking that fires before or regardless of it. Flag it and route the legal
   question onward rather than reasoning about the regulation yourself.
6. **No tracking plan.** Events accumulated by whoever needed one, documented
   nowhere. Propose the compact table — event, when it fires, key properties,
   which funnel step it serves — because that document is what keeps the next
   twenty events consistent.
7. **Duplicate and double-fired events.** The same moment captured by two names
   from two layers, or a capture in a component that re-renders, inflating every
   number derived from it. The tell is a count that cannot be reconciled with the
   database.

## How to verify before you claim

- **Read the wrapper.** Most codebases funnel captures through one helper, and
  that helper is where naming, default properties and redaction belong — a fix
  applied there covers every call site, which is almost always the better
  proposal.
- **Check the environment gating.** Events fired from local and preview
  environments into the production project make every number wrong; a filter or
  an environment property is the cheap fix.
- **Confirm the property is actually sensitive before calling it PII.** An
  opaque internal id is not an email. Say which field and why.
- **Propose the migration, not just the rule.** A naming convention that leaves
  existing events unrenamed splits every metric in two. Say what happens to the
  history: rename and lose continuity, or alias and keep it.
