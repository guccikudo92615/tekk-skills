---
name: grill-to-loop
description: 'Interview the user about a problem that keeps happening to them, then build a Tekk loop that watches for it — reading what their loops already cover first, so an existing loop gets aimed rather than duplicated. Use when someone wants to build or create a loop, wants Tekk to "watch for" or "keep an eye on" something, or describes a recurring problem they are tired of finding by hand.'
allowed-tools:
  - mcp__tekk__list_loops
  - mcp__plugin_tekk_tekk__list_loops
  - mcp__tekk__get_loop
  - mcp__plugin_tekk_tekk__get_loop
  - mcp__tekk__create_loop
  - mcp__plugin_tekk_tekk__create_loop
  - mcp__tekk__configure_loop
  - mcp__plugin_tekk_tekk__configure_loop
  - mcp__tekk__run_loop
  - mcp__plugin_tekk_tekk__run_loop
---

Build me a loop. Something keeps happening to this team and nobody is watching for it — interview me about it, then write the loop and put it in my library.

A loop is not a reminder and not a linter. It wakes on its own, reads this codebase and whatever sources are connected, and files at most one proposal per run for a person to accept or decline. So the whole job here is to find out what it should be looking FOR, precisely enough that a run can tell a finding from a normal Tuesday.

**You are the better interviewer for facts.** You have this repository open and, most likely, their other tools connected. Anything you can look up, look up — never ask a question you can answer yourself. Ask only what is in the user's head.

## 1. Read the state before you ask anything

- `list_loops` — what already runs here, what is on, what each one is for. **Do this first, every time.**
- `get_loop` on anything that sounds close to what they are describing.
- Look at the repo: where does the thing they are complaining about actually live?
- If a source they will need is not connected, you will find out at create time — but knowing now saves a round.

## 2. Say what you see, and offer the cheaper answer first

Tell them, in two or three lines, what is already covering this ground.

**If an existing loop overlaps the ask, say so and offer to AIM it instead.** Most "I wish Tekk watched for X" is a loop that already reads those files and was never told to care about X. Aiming one is instant, costs nothing, and does not add a thing to maintain. Building a fresh loop is right when:

- no existing loop reads that source at all, or
- the method is genuinely different from what any of them do, or
- they hear the offer and still want their own.

Take their answer. If they want the aim, set it and stop — that is a good outcome, not a failed one.

## 3. Grill, in rounds

Ask **the whole answerable frontier at once** — every question whose answer does not depend on another answer — and give a **recommended answer for each**, so the user can say "yes, yes, no — it is actually the webhook path" instead of composing five replies. Then take what they said, work out what it opened up, and ask the next round. Stop when the frontier is empty, not when you hit a question count.

Round one is usually:

1. **What exactly keeps happening?** Not the category — the event. "Webhooks stop being delivered after a deploy", not "billing problems".
2. **How do you find out today?** That names the evidence: a Sentry error, a customer email, a dashboard nobody opens, a number that quietly drops.
3. **Where does it live in the code?** Confirm what you already found rather than asking blind.
4. **What should come out of a run?** A proposal that says "here is the gap, here is the fix"? Or should Tekk also DO something — post to a channel, create an alert?
5. **What would make you angry to be told?** The false-positive shape. This is the most useful question in the interview and the one nobody volunteers.

**Sharpen fuzzy language before you build on it.** If they say "checkout", ask which — the Stripe webhook path, or the form before it. A word that means two things will produce a loop that watches neither. (This rule and the round format are borrowed from Matt Pocock's `grilling` and `domain-modeling` skills, MIT.)

**Never ask about Tekk's own vocabulary.** What a diff means to the loop, how its titles should read, when it is worth running — you write those, from the answers above. If you find yourself about to ask "should this be subject, suspect or intent?", answer it yourself.

## 4. Write it, show it, create it

A loop is a **manifest** plus a **shelf of skills**. Write both.

**`loopMd`** is the loop's identity, injected on every run: what it is for, the seams it cares about, what counts as a finding and what explicitly does not. Write the false-positive answer into it as a rule. Do NOT restate generic discipline — "never guess", "one proposal per run", "cite your evidence" — every loop already receives all of that, and repeating it crowds out the part that is actually about this loop.

**The shelf** is one or more skills, and exactly ONE is opened per run — whichever owns the most changed files. One skill is correct for a narrow loop. Use several when the loop really has distinct methods (how you audit delivery is not how you audit retries), and order them specific-first, with the most general last. Each skill is a `SKILL.md`:

```
---
name: webhook-delivery
description: Audits whether webhook events still reach their handlers. Use when billing or webhook code changed.
---

# webhook-delivery

What to evaluate: ...
How to verify before you claim it: ...
```

That `description` is what the run reads to decide which skill to open, so write it as a when-to-use, not a title. Bundle reference material with `files` when a skill needs a checklist or a table.

**Show the user what you wrote** — the job in one line, the sources, the shelf with each skill's one-line purpose — before you call anything.

Then `create_loop`. If it comes back with errors, they are specific and actionable: a source that is not connected on this project, a tool that is not read-only, a skill with no front matter, or a job too vague to run ("make the app better" is refused on purpose — a loop like that files something generic every night forever). Fix and call again; do not relay a validation error to the user as if it were a dead end.

**Writes are Actions, not tools.** A loop never gets a tool that changes anything. If it should post to Slack or create an alert, declare that as an action — Tekk performs it when a person accepts the proposal, under a grant the workspace controls. Say that plainly if they ask why their loop cannot "just fix it".

## 5. Close with the settings, before and after

The loop exists and is OFF. Show one block — what it is now, what you propose, in that order — and apply it only once they say yes:

```
stripe-drift            now            proposed
  enabled               off            on
  autonomy              —              ask (it proposes; you decide)
  runs at most every    —              48h
  aimed at              —              "we just moved to Stripe Connect;
                                        watch the connected-account path"
```

Then apply: `configure_loop` for enabled, autonomy and cadence — **owner-only**, so if this user is not an owner, mark those lines "needs an owner" and say who to ask rather than failing halfway. The aim is a separate, anyone-can-set instruction; if this workspace has no way to set one yet, say so and move on.

Finish by telling them what happens next, in one line: **the first run asks before it runs.** Whatever the autonomy setting, a loop that has never produced a run anyone has read gets one card, once, with its estimated cost on it — and the same after every edit. `run_loop` returns `pitched`; greenlighting it runs.

## What a good loop looks like when you are done

- Its job names a **concrete failure class** and where it shows up.
- Its `loop.md` says what is NOT a finding.
- Its shelf has as many skills as it has genuinely different methods — usually one or two.
- Its sources are ones this project actually has connected.
- The user could explain, in their own words, what it will do tonight.
