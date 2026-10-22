## Tekk

Tekk is this project's AI CTO. Its loops audit the codebase in the background,
raise what they find as proposals, and keep the spec board. This repository is
connected to the **{{workspaceSlug}}** workspace through the `{{mcpServer}}` MCP
server ({{appUrl}}). Never ask which workspace — the connection decides it.

Start here:
- `/sweep` — work the whole pending-proposal inbox in one pass.
- `/blitz` — turn planned specs into parallel coding sessions.
- `/triage` — reconcile the spec board with what actually merged.
- `/spec` — write a spec the board can actually close.
- `/ideate-loops`, `/grill-to-steer`, `/steer-loops` — choose, aim and run the loops.

Without the skills installed, the same commands are MCP prompts:
`/mcp__{{mcpServer}}__sweep`, `/mcp__{{mcpServer}}__blitz`, and so on.

Rules when working here:
1. Begin with `get_workspace_overview`. Its `drift` block lists specs whose
   status disagrees with what merged — believe drift over a bare status.
2. Read a spec with `get_spec` before implementing it, and move it to
   `in-progress` with `update_spec_status` the moment you start.
3. A spec closes only from a merged PR whose body says `Closes <SPEC-ID>`, over
   a ticked `## Build order`. Tick what you finished (`update_spec`) and write
   the closing line yourself — a PR that merely mentions a spec moves nothing.
4. Never claim a session or spec completed by hand; completion is derived
   from the merged closing PR.
5. Research the code before `create_spec`: it requires load-bearing claims
   with real locators (file:line, a URL, a measurement).
