---
name: steer-loops
description: 'Run the loops rather than watch them: see what is on and what it costs, fire one now or schedule a group, work the pitch queue, and set cadence and autonomy. Use when the user asks why a loop has not run, wants one to run more or less often, wants to turn one on or off, asks what the loops are costing, or has pitches waiting on a decision. Aiming loops at a topic is grill-to-steer; this is the operational half.'
allowed-tools:
  - mcp__tekk__list_loops
  - mcp__plugin_tekk_tekk__list_loops
  - mcp__tekk__list_runs
  - mcp__plugin_tekk_tekk__list_runs
  - mcp__tekk__get_run
  - mcp__plugin_tekk_tekk__get_run
  - mcp__tekk__list_pitches
  - mcp__plugin_tekk_tekk__list_pitches
  - mcp__tekk__decline_pitch
  - mcp__plugin_tekk_tekk__decline_pitch
  - mcp__tekk__veto_pitch
  - mcp__plugin_tekk_tekk__veto_pitch
  - mcp__tekk__spend_report
  - mcp__plugin_tekk_tekk__spend_report
  - mcp__tekk__list_spend
  - mcp__plugin_tekk_tekk__list_spend
  - mcp__tekk__configure_loop
  - mcp__plugin_tekk_tekk__configure_loop
  - mcp__tekk__configure_workspace_loops
  - mcp__plugin_tekk_tekk__configure_workspace_loops
  - mcp__tekk__set_loop
  - mcp__plugin_tekk_tekk__set_loop
---

Steer the loops — they are configurable, not just observable. Everything the loops settings screen does is here, and an answer to "why hasn't it run?" is three calls away.

This is the **operational** half: what is on, what it costs, what runs when. Aiming a loop at a subject — what it should care about — is `grill-to-steer`, which writes a steer the runs read. If the user wants better findings, that is the other skill. If they want *different scheduling, spend or state*, stay here.

## Ground rules

1. **Money is real and some of these calls spend it.** `greenlight_pitch` defaults to `when: "now"`, which starts the run **immediately** — the same thing the web app's Run-now button does. Say so and get a yes before calling it. `when: "tonight"` schedules it for the workspace run hour instead and is the safe default when nobody asked for it right now.
2. **Autonomy does not cause runs.** It decides what happens to a run already proposed: `ask` waits for a tap, `auto` schedules it for the workspace night and stays vetoable all day, `full` runs it immediately. Setting autonomy alone schedules **nothing**. Telling a user "it will run nightly" after changing only autonomy is a falsehood they will act on.
3. **`configure_loop` owns the loop; `set_loop` does not.** `configure_loop` carries `enabled`, `autonomy`, `agendaFloorHours`, `visibleProjectIds` and `applyToAllProjects`. `set_loop` toggles premise checking and nothing else — reaching for it to enable a loop fails validation.
4. **Read before you write.** `list_loops` first, always. Half of "why hasn't it run" is a loop that is off, and the other half is a cadence nobody set.
5. **Loops bill the workspace owner**, not whoever triggered them. "What is this costing?" and "who is paying?" are different questions.

## Seeing

- **`list_loops`** — which loops are on, how much rope each has, how often it must run. The picture everything else starts from.
- **`list_runs`** (filter by `loop`, `limit`) — what actually happened.
- **`get_run`** with a `runId` — one run in full: what it examined, what it filed, what it cost. `detail` opens the brief.
- **`spend_report`** over a `since`/`until` window, groupable by `loop` or project — dollars and credits, and whose credits paid. **`list_spend`** is the rows behind it.

## Running

- **`run_loop`** with a `loop` and `projectId` — fire one now, rather than waiting for a wake.
- **`schedule_loop_runs`** — a group at once: `runs` is a list of `{loop, projectId}`, `at` picks the time (omit for tonight). **Use `dryRun` first** and show the user what would be scheduled. Scheduling five runs the user did not want is the expensive mistake here.

## Deciding — the pitch queue

A pitch is the gap between a loop *wanting* to run and the run happening.

- **`list_pitches`** — `pitched` is waiting on the human; `scheduled` will go at the workspace run hour and can still be stopped.
- **`greenlight_pitch`** — accept one. `when: "now"` spends immediately; `when: "tonight"` schedules it. See ground rule 1.
- **`decline_pitch`** — say no, and the reason steers what later wakes propose. A bare no teaches nothing.
- **`veto_pitch`** — cancel a scheduled run before it starts.

## Changing

- **`configure_loop`** — `enabled` on or off; `autonomy` (`ask` / `auto` / `full`, or null to inherit); `agendaFloorHours` for cadence; `visibleProjectIds` for the sibling projects it may read; `applyToAllProjects` to write enabled and autonomy across every project at once. Owner-only.
- **`configure_workspace_loops`** — the workspace **defaults** loops inherit: `autonomyDefault` and `nightlyHourLocal`. It does not touch a loop that has its own override, so never report it as having reconfigured everything.
- **`set_loop`** — premise checking for one loop. That is all it does.

### Cadence, precisely

`agendaFloorHours` is how stale a loop may get before it **must** be pitched — 24 means nightly. Pair it with autonomy `auto` and that pitch is scheduled for the night and stays vetoable through the day.

It is an **eligibility** floor, not a promise. Leaving it null does not mean no cadence: it inherits a default, longer while the workspace is quiet. And an overdue loop is still held back while nobody is reading the inbox, while too many of its proposals sit undecided, or when it has no repository to read. Only loops that read a repository diff accept a floor at all.

So: "this makes it eligible nightly" — never "this will run nightly."
