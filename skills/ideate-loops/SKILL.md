---
name: ideate-loops
description: 'Work out which loops are worth building for this project: read what is connected and what the existing loops already cover, ask what keeps hurting, and come back with a short ranked list — each one labelled as a new loop, an aim on a loop they already run, or a connection that would unlock it. Use when someone asks what loops they should have, what Tekk could watch for them, what to do with a tool they just connected, or says they do not know where to start.'
allowed-tools:
  - mcp__tekk__list_connections
  - mcp__plugin_tekk_tekk__list_connections
  - mcp__tekk__list_loops
  - mcp__plugin_tekk_tekk__list_loops
  - mcp__tekk__list_loop_ideas
  - mcp__plugin_tekk_tekk__list_loop_ideas
---

Work out what Tekk should be watching on this project that it is not watching yet. Do not build anything — end with a short list the user can choose from.

There are thousands of possible loops across a handful of connected tools, which is exactly why nobody can picture one. Your job is to turn that space into three to seven ideas this particular team would actually want, and then hand the one they pick to the interview that builds it.

**You write nothing.** No `create_loop`, no settings. The output is a shortlist and a hand-off.

## 1. Read before you ask

- `list_connections` — what this project has connected and what each one can read. Connections belong to a project, not a workspace, so this is the real inventory.
- `list_loops` — what already runs here and what is switched on.
- `list_loop_ideas` — seeded ideas whose sources are all connected, plus what one more connection would unlock. These are seeds, not a menu: combine them and go past them.
- `list_proposals` and recent runs — what the loops have actually been finding, and what the user has been rejecting. A loop whose findings keep getting declined is a signal about this team's taste, not a loop to build more of.
- The repo itself: what is this product, and which parts of it look least watched?

## 2. Say what you see, in four lines

Before any question, tell them what you found — it is usually the most useful thing in the whole conversation:

- what is **connected and barely used** ("Datadog is connected; nothing reads it"),
- what is **already covered** ("payments and reliability both read the billing code"),
- what has **never been looked at** (a surface no loop's scope includes),
- what the loops have been **finding and getting rejected**, if there is a pattern.

## 3. Ask what hurts — one round

Ask the whole answerable frontier at once, with a recommended answer on each, so they can say "yes, yes, no — it's the onboarding flow". Do not interrogate; three or four questions is the whole round.

- What did you last find out too late?
- Which part of the product do you feel you cannot see into?
- What do you check by hand, on a rhythm, because nobody automated it?
- What breaks in a way that only shows up days later?

If a loop they already run has its own opening question, use it where the conversation goes near that loop — not as a list to march through. (The round format is borrowed from Matt Pocock's `grilling` skill, MIT.)

**Anti-roster rule:** never read them the catalog. A list of twenty ideas is the same problem they came with. Three to seven, chosen because of what they just told you.

## 4. Combine, rank, label

For each idea on the shortlist, four lines:

```
Deploy → regression watch                          AIM on `reliability`
  The problem   You deploy and find out later that it broke something.
  Sources       Sentry (connected)
  Wakes on      A deploy.
  Proposes      What got worse after it, tied to what changed.
  Why you       Three of your last five rejections were "already fixed" —
                you are finding these by hand, a day late.
```

Every idea carries a **kind**, and the label is what keeps this honest:

- **AIM on `<loop>`** — a loop you already run reads that code and was never told to care about this. Instant, free, nothing new to maintain. **Most good ideas are aims; lead with them.**
- **NEW loop** — nothing you run covers this source or this method.
- **CONNECT `<tool>`** — the idea is good and needs something not connected yet. Say what it would unlock, once, as an answer to a problem they stated — not as a nag.

Rank by what they told you hurts, not by how clever the combination is. Say plainly if you think one of them is not worth building.

## 5. Hand off the one they pick

- **A new loop** → start `grill-to-loop` with the idea as the opening ask ("they want a loop that watches for X, joining Y and Z; here is what they said about it"). Do not build it here — that interview exists because a loop that skips it comes out vague.
- **An aim** → set it on that loop, in their words, and confirm what it will change.
- **A connection** → tell them what to connect and what it unlocks, then stop.

If nothing on the list appeals, that is a real answer. Say what you would watch for instead, and leave it.

## What a good session leaves behind

- They can name the one thing they most want watched, which they probably could not at the start.
- At least one idea was an aim, not a build.
- Nothing was created that they did not choose.
