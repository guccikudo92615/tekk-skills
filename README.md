# Tekk

Drive your [Tekk](https://tekk.coach) board from the coding agent you already use. 5 [Agent Skills](https://agentskills.io) and the Tekk MCP server, in one install.

> **Generated — do not edit here.** Every file is built from the Tekk repo and pushed on each release. A fix belongs upstream; an edit made here is overwritten by the next push. Issues and discussion are welcome.

## Install

In Claude Code — this also connects the MCP server, so there is nothing else to wire up:

```
/plugin marketplace add guccikudo92615/tekk-skills
/plugin install tekk@tekk
```

For Codex, Cursor and the other agents the skills CLI targets, take the skills and add the MCP server yourself:

```bash
npx skills add guccikudo92615/tekk-skills -a codex
```

Then in `~/.codex/config.toml`:

```toml
[mcp_servers.tekk]
url = "https://app.tekk.coach/api/mcp"
bearer_token_env_var = "TEKK_API_KEY"
```

You need a Tekk account: these skills act on your board.

## What you get

| Skill | What it does |
| --- | --- |
| [`sweep`](skills/sweep/SKILL.md) | Work the whole pending-proposal inbox in one pass: re-check every finding against the code as it is now, say which no longer hold and whether that means fixed or merely moved, then decide the batch in one call once the user says go. Use when proposals have accrued, or the user asks what their loops found. |
| [`blitz`](skills/blitz/SKILL.md) | Turn what the loops raised and what is already on the board into parallel coding sessions: cluster into the biggest collision-safe batch, open one session per spec, and register each so the fleet stays visible. Use when the user wants to start their planned work, parallelize, or "run through everything". |
| [`triage`](skills/triage/SKILL.md) | Resolve the whole spec board against git reality in one pass — find specs that shipped but were never closed, were marked started but never begun, duplicate each other, or target code that no longer exists. Use when the board is untrusted ("what is actually done?", "clean up the specs"), before planning or dispatching from it, and after a batch of PRs merges. Operates on specs; the pending-proposal inbox is sweep. |
| [`spec`](skills/spec/SKILL.md) | Write a spec the board will accept and the drift gate can actually close — research the claims first, keep every checkbox finishable by the coding agent, and write acceptance as prose rather than boxes. Use when creating or rewriting a spec, turning a conversation into tracked work, or when a spec has stalled because nobody can tick its checklist. |
| [`grill-to-steer`](skills/grill-to-steer/SKILL.md) | Interview me about my product, then aim my autonomous loops at what I actually care about: read where my loops stand, propose what is worth digging into, grill me in rounds, and store the result as a steer my runs read. Use when I want to aim or steer my loops, ask what my loops should be looking at, say I am about to ship or launch MY PRODUCT (not a coding session, deploy or dev server), or ask to be grilled about my product. |

The board is what these act on: Tekk runs review loops over a connected repository and raises what it finds as proposals. `sweep` works that inbox, `blitz` turns the result into parallel coding sessions, `triage` keeps the board honest against git, `spec` writes work that can actually be finished, and `grill-to-steer` interviews you about your product and aims the loops at what you answer.

## Licence

MIT.
