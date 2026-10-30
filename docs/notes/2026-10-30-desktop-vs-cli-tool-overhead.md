# Investigation candidate: Claude Code desktop app vs CLI tool-surface overhead

Date: 2026-10-30. Status: candidate for the M0 inventory (Toolset & Harnesses row, Claude Code session transcripts and usage logs). Not scheduled; no result yet.

## Observation

The Claude Code desktop app registers internal MCP servers that the terminal CLI does not. Their tool schemas are sent with every request and show up in `/usage` as MCP entries:

- `ccd_turn`: `update_plan`, `wrap_up`
- `ccd_session`: `dismiss_task`, `hand_off_to_session`, `mark_chapter`, `read_widget_context`
- `Claude_Browser` (built-in browser pane): `browser_batch`, `computer`, `find`, `form_input`, `get_page_text`, `javascript_tool`, `navigate`, and others

Evidence on 4070pc: the tool roster saved in a desktop session's subagent prefix (`~/.claude/projects/C--Users-Matt-Desktop-MyDocs/10598c9b-7261-416f-befb-58261132b9b8/subagents/*.prefix.json`) lists `mcp__ccd_turn__*` and `mcp__ccd_session__*` next to the `Claude_Browser` tools. None of these names appear in `~/.claude/settings.json`, `~/.claude.json`, any `.mcp.json`, or the plugin directories, so they are app-provided, not operator-configured. The expansion of `ccd` to "Claude Code Desktop" is an inference from the names, not documented.

## Why it matters to Tuan

Two sessions running the same request on the same host can carry a different fixed per-request context cost depending on whether the launcher was the desktop app or the CLI. If Tuan's records do not capture the launcher surface, that difference will be attributed to the task, the model, or the context strategy. It also changes what the model is able to do mid-turn (plan widget, wrap-up, browser pane), so outcome and effort comparisons across surfaces are not like-for-like.

## What to measure

1. Per-request schema overhead for each surface: take the first request of a fresh session on the desktop app and on the CLI with identical settings and MCP configuration, and compare cached-input and input token counts. Record the tool roster of each.
2. Whether the surface is recoverable from existing session transcripts (the `prefix.json` roster, or a field in the session JSONL), so it can be a derived feature rather than operator-supplied.
3. Whether the desktop-only tools change behaviour in a measurable way: frequency of `update_plan`/`wrap_up` calls, and any difference in turn count or retries on matched tasks.

## Open questions

- Which surface Matt uses for which kinds of work, and whether that correlates with task type in a way that would confound comparisons.
- Whether Codex has an equivalent split (desktop/IDE extension vs CLI) that needs the same field.
- Whether the overhead is large enough to matter against typical startup context here (measured about 93k tokens before the first tool call in the MyDocs workspace, per `MyDocs/AGENTS.md`).

## Proposed contract change

Add `launcher_surface` (values such as `claude-desktop`, `claude-cli`, `codex-cli`, `unknown`) to the run or session record in `docs/contracts.md` when M0 drafts it, with provenance marked native, derived, or operator-supplied per the M0 field-mapping rule.
