# Hook design — event constraints, type selection, pattern library

Read this only when the guardrail-depth interview answer is (b) scripted hook, (c) LLM-judgment hook, or (d) resolved to one of those after seeing the task.

## Hard constraint

Subagent-frontmatter `hooks:` supports only three events — **`PreToolUse`, `PostToolUse`, `Stop`** — not the full Claude Code hook catalog (no `UserPromptSubmit`, `SessionStart`, etc. at agent scope). Map the user's intent to one of these:

- Prevent an action before it happens → **`PreToolUse`** (blockable: can `deny` with a reason, or rewrite `updatedInput`).
- Validate or react after an action succeeds → **`PostToolUse`** (can't undo, but can add `additionalContext` or flag via `permissionDecision` on a subsequent failure).
- Clean up or summarize when the subagent finishes → **`Stop`** (fires as `SubagentStop` in this context).

Re-confirm this constraint against Phase 2's live schema check — it's version-gated and could change.

## Hook type — pick the cheapest one that does the job

| Type | Use for | Latency | Notes |
|---|---|---|---|
| `command` | Deterministic checks (block a bash pattern, run a linter) | ~100ms | Default choice |
| `prompt` | Semantic judgment a regex can't express | ~2-5s | Set `model` explicitly if the fast default isn't enough |
| `agent` | Multi-step verification | ~5-10s | Experimental; reserve for genuinely complex checks |
| `http` / `mcp_tool` | Call an existing external service/MCP tool | varies | Only offer if the user already has such a service — never invent one |

## Command hook scripts

- Write scripts to `.claude/hooks/<agent-name>/<hook-name>.sh`.
- Invoke as `bash "${CLAUDE_PROJECT_DIR}/.claude/hooks/<agent-name>/<hook-name>.sh"` — an explicit interpreter prefix, not a bare path — so the script never needs the executable bit. `Write` alone can create it; no `Bash`/`chmod` required.
- Parse hook input JSON with `jq -r` and quote every variable. Never interpolate `tool_input` fields directly into a shell command string — that's command injection into your own hook.

## Pattern library — fill in the blanks, don't freehand a new script each time

1. **Block a destructive Bash pattern** — `PreToolUse`, `matcher: "Bash"`, `if: "Bash(rm *)"` (or the specific pattern from the interview's Bash follow-up), deny with a clear reason.
2. **Enforce lint/format after edits** — `PostToolUse`, `matcher: "Edit|Write"`, runs the repo's existing lint/format command — detect it from `package.json`/`pyproject.toml`/etc., don't invent one.
3. **Restrict edits to a path prefix** — `PreToolUse`, `matcher: "Edit|Write"`, deny if `tool_input.file_path` falls outside an allowed prefix.
4. **Audit-log every tool call** — `PreToolUse`, `matcher: "*"`, appends to a log file. Only offer this if the user explicitly wants an audit trail — it adds latency to every single call.

## Trust caveat

Project-level `.claude/agents/*.md` hooks require workspace trust. State this plainly in the Phase 4 summary whenever a `hooks:` block is written: "This agent's hooks require the workspace to be trusted; if this folder isn't trusted yet, Claude Code will prompt for it."
