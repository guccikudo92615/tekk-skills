---
name: evals
description: 'Audits whether a workspace can change a prompt, model or retrieval setting and KNOW it improved things, and whether production misbehaviour is diagnosable. Use when reviewing eval harnesses, golden sets, graders, judges, traces or LLM observability — or when nothing measures quality at all.'
---

# Skill: evals

**Skill:** measurement — can this workspace change a prompt, a model, or a
retrieval setting and KNOW the change made things better? And when the agent
misbehaves in production, can anyone see why? Find the core AI path first (the
one the product's value rides on); everything in this lane is judged relative
to it.

## What to evaluate

1. **Regression gates.** Golden sets, model-graded checks, run/fail
   thresholds wired into CI or a pre-deploy step — or vibes? The archetype
   finding: the core path has no gate, so every prompt tweak is a blind bet.
   Grade the gate's substance, not its existence — five happy-path cases on a
   side feature is "partial", not "covered".
2. **The eval data itself.** Does the golden set reflect what real users
   actually send (hard cases, adversarial input, the long tail), or only what
   the author imagined? Is it versioned alongside the prompts it protects, so
   a prompt change and its eval change travel together?
3. **Model-graded checks.** If an LLM grades outputs: does the grader have a
   rubric with anchored examples, or a bare "rate 1-10"? Unanchored scores
   drift and can't gate anything.
4. **Observability.** When a user reports a bad answer, can they find that
   call? Traces with per-call attribution (which prompt version, which model,
   which tools fired, what came back), structured logs on the AI path — or
   `console.log` and guesswork? A misbehaving agent nobody can inspect stays
   misbehaving.
5. **The feedback loop.** Do production failures flow back into the eval set,
   or is the same class of failure rediscovered forever?

## How to triage

An absent gate on the core path outranks everything else in this lane. After
that, prefer the change that makes the *next* incident diagnosable (tracing on
the AI path) over broader-but-shallower coverage. Concrete beats aspirational:
"add these 12 real failures as regression cases, gate deploys on them" ships;
"build an eval culture" doesn't.
