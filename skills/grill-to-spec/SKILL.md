---
name: grill-to-spec
description: 'Turn an idea into a spec that is ready to build: read the board and the code first, grill me in rounds until no decision is left open, play the result back, and only then write the spec, or a parent with sub-tasks, under the same rules as write-spec. Use when I have an idea, a feature or a problem but not yet a plan, say "grill me" about something I want built, or want a spec and the decisions behind it are still open.'
allowed-tools:
  - mcp__tekk__get_workspace_overview
  - mcp__plugin_tekk_tekk__get_workspace_overview
  - mcp__tekk__list_specs
  - mcp__plugin_tekk_tekk__list_specs
  - mcp__tekk__get_spec
  - mcp__plugin_tekk_tekk__get_spec
  - mcp__tekk__list_epics
  - mcp__plugin_tekk_tekk__list_epics
  - mcp__tekk__create_spec
  - mcp__plugin_tekk_tekk__create_spec
  - mcp__tekk__update_spec
  - mcp__plugin_tekk_tekk__update_spec
  - mcp__tekk__add_dependency
  - mcp__plugin_tekk_tekk__add_dependency
---

Grill me about something I want built, then write it down as a spec my coding agent can build without a meeting.

The reason this is an interview and not a form: a spec that exists is implementation-ready. Every decision in it has to be made **before** it is written, because an open decision cannot be a checkbox (nobody can tick "decide the shape"), and a decision a coding agent makes alone is a guess that looks like a plan. So the job is to find every decision this work hangs on, put each one to me, and write nothing until none is left.

**You are the better interviewer for facts; I am the only source of decisions.** You have the repository open and the board connected. Anything you can look up, look up, and never ask me a question you could answer yourself. Ask only what is in my head: who it is for, what matters, what I would accept, what I would refuse.

## 1. Read before you ask anything

- `get_workspace_overview` for the shape of the board: what is in flight, what is drifting.
- `list_specs`, searched by the words I used and by the area of code it would touch. **If a spec already covers this, say so first** and offer to sharpen that one with `update_spec` instead of adding a second. A duplicate spec is worse than none: two agents build two versions.
- `get_spec` on anything close, so you know what it already decided.
- `list_epics` if the work plainly belongs to a larger initiative.
- **The code.** Find where this would live, what it would change, and what already exists that it could reuse or would break. If your harness can run a sub-agent, send one to explore while you prepare the first round. Exploration still running is an unsettled fact: hold back only the questions that depend on it, and ask the rest now.

Then tell me, in three or four lines, what you found: where this lives today, what is already on the board near it, and anything in the code that changes the question. That is the ground the interview stands on.

## 2. Grill, in rounds

Map the work as a **design tree**: every decision branches into the decisions that hang off it. The **frontier** is every decision whose prerequisites are already settled, the questions you can ask now without guessing at answers you have not heard. Ask the whole frontier in one round, number each question, and give your recommended answer to each, so I can reply "yes, yes, no, it is the admin view" instead of writing five paragraphs.

Format a round like this:

```
❓ **Q1** - **<question title>**: <the question, with the options if there are any, and what you found in the code that bears on it>

➡️ <your recommended answer, and the one-line reason>

---

❓ **Q2** - **<question title>**: <...>

➡️ <...>
```

Each answer reshapes the tree. Recompute the frontier and ask the next round. A question whose answer depends on another question still open in this round belongs to a later round.

### The branches every spec has

Work these until each is settled. They are the sections the spec will need, so a branch left open is a hole in the spec.

1. **Who is hurt today, and how?** Not the feature, the person and the moment it goes wrong for them. This becomes the Why, and the board refuses a spec without one.
2. **What becomes possible once it ships?** And what happens if it is never built. If the honest answer is "not much", say so: that is the most useful thing this interview can find.
3. **What exactly is in, and what is out?** Name the edges. Everything named out goes under Out of scope, where it survives instead of creeping back in.
4. **The behaviour decisions.** Every place where two reasonable builds would differ: what happens on the empty state, on failure, on the second click, for the user without permission, for existing data. Recommend from the code; ask only where the code does not decide.
5. **How will we know it works?** An observable outcome, a test, a command. This becomes Acceptance, written as prose, so phrase the answers as things a person could check.
6. **One spec or a tree?** Recommend this yourself from what you found. A small, indivisible change is one spec with an ordered build order. Independently buildable parts are a parent with sub-tasks, with the order between them as dependencies. Only ask me when the split changes what ships first.

### Rules for the interview

- **Sharpen fuzzy language before you build on it.** If I say "the dashboard", ask which one. A word that means two things produces a spec two agents build differently.
- **Never ask me about Tekk's own vocabulary.** The spec type, priority, `touches` globs, how the build order is phrased, whether it needs a picture: you decide those from the answers.
- **A decision I cannot make yet is not a question for this spec.** Say so, and either leave it out of scope or offer it as its own spec. Do not write it in as a checkbox.
- **"Measure first" and "watch it for a week" are follow-ups.** An agent cannot wait a week. If the right answer is to learn something before building, that learning is a separate research spec, and this one waits on it.
- **Record what I turned down, and why.** Every rejected option goes under Decisions with its reason. An unrecorded rejection gets re-proposed in six months.
- **Push back.** If an answer contradicts the code, or another answer, or makes the work much bigger than its Why, say so in the next round rather than writing it down.

The interview is done when the frontier is empty: every branch visited, nothing silently assumed.

## 3. Play it back, then wait

Before anything is written, show me one block:

- **Summary**: two to four plain sentences, for someone who was not here.
- **Why**: who is hurt, what it unlocks, what happens if it is never built.
- **Decisions**: each one, with the options turned down.
- **Shape**: one spec, or the parent and its sub-tasks in build order with what each depends on.
- **Build order**: the steps, each one something a coding agent finishes in the pull request.
- **Acceptance**: how we will know, in prose.
- **Out of scope**.

Ask whether this is what I meant. Write nothing until I say yes. If I correct something, fix it and show only what changed.

## 4. Write it

Write it under the same rules as the `write-spec` skill; if that skill is installed, read it now and follow it. The rules that matter most:

- The body opens with `## Summary` and `## Why`, and the Why stands alone. A sub-task restates the purpose in its own words and never points at its parent instead.
- Every load-bearing claim about the code goes in `assumptions` with a locator someone else can check (`path/to/file.ts:123`, a URL, a measurement with its source). Facts you found during the interview belong here, not in my memory.
- **Every checkbox is a build step the coding agent finishes in the pull request.** No decisions, no waiting, no "verify", no "run the tests", no work in another tool. Those were settled in the interview or belong in Acceptance.
- `## Acceptance` is prose bullets, never boxes.
- `## Decisions` carries every rejected option and its reason.
- Declare `touches`, the file globs the work will change.

For one spec, call `create_spec`. For a tree, create the parent first, then each sub-task with `parentSpec` set and `dependsOn` naming the sub-tasks it waits for (or `add_dependency` afterwards). One feature is one tree, never a row of sibling specs and never an epic of its parts.

If the board refuses a body, the refusal names the line and what it should be instead. Fix it and call again; do not hand me a validation error.

## 5. Close

Tell me what now exists, by id, with the one-line purpose of each. Then offer the next step as a question, without starting it: build it now with `/blitz`, or leave it on the board for the night shift.

---

The round structure (the design tree, the frontier, the numbered questions with recommended answers, finding facts rather than asking for them) is adapted from the `grilling` skill in Matt Pocock's `mattpocock/skills` repository, MIT licensed. The "sharpen fuzzy language" discipline comes from its `domain-modeling` sibling.
