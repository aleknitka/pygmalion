# Output spec — frontmatter contract & write checklist

Confirm every field below against Phase 2's live schema check before using it — this file records the design intent, not a guaranteed-current field list.

## Write location

Determined by the interview's agent-level answer, not assumed:
- Project → `.claude/agents/<name>.md`
- User → `~/.claude/agents/<name>.md`

Run the Phase 0/pre-write collision `Glob` against that same chosen directory (not both, unless the user is deciding between levels for a name already taken at one of them).

## Required fields

- **`name`**: kebab-case, lowercase letters and hyphens only. No leading hyphen. No colon (colons are reserved for plugin-scoped ids like `plugin:agent`). Check for a collision via `Glob` on the chosen level's agents directory before writing — a real collision is handled in Phase 0, not silently overwritten here.
- **`description`**: plain text describing when Claude should delegate to this subagent. Phrase as "Use when X" (plus "not for Y" if an easy confusion exists) so automatic delegation matches correctly — mirror the same pattern used in this skill's own frontmatter.

## Core optional fields

- **`tools`**: comma-separated allowlist built from the interview's tool-access answer. Default to an explicit minimal list; only omit entirely if the user explicitly wants the subagent to inherit every tool.
- **`model`**: `haiku` / `sonnet` / `opus` / `inherit`, from the interview.

## Fuller field set — apply by inference, state reasoning in the final summary; escalate to a question only if genuinely ambiguous

- **`disallowedTools`**: from the Phase 1 Bash follow-up, if one was given.
- **`permissionMode`**: omit by default. Only ask about it if guardrail depth was (b)/(c) *and* the agent also holds `Write`/`Edit`/`Bash` — i.e. the tool allowlist alone isn't already a strong guardrail.
- **`maxTurns`**: propose a sane default (e.g. 15) for open-ended or loop-prone tasks; omit for narrow single-shot tasks. Always adjustable at the confirm step.
- **`skills`**: from Phase 2's local Skills inventory. Include automatically on a clear match; otherwise, if the domain warrants it, follow `skill-packaging.md` to author a new companion Skill instead of duplicating that knowledge inline in the system prompt.
- **`isolation: worktree`**: propose it (don't silently apply) when the agent holds broad `Write`/`Edit`/`Bash` access and the task implies repo-wide changes.
- **`memory`**: only ask about this if the task description itself implies recurring, stateful work across sessions (e.g. "track known issues over time").
- **`color`, `effort`**: choose without asking — cosmetic/tuning fields with no correctness impact.
- **`hooks`**: see `hooks.md`. Only present if guardrail depth resolved to (b) or (c).

## System prompt body

- Single responsibility: explicit inputs, outputs, and a done-condition.
- No restating obvious platform behavior (don't explain what `Read` does).
- Bake in any guardrail language the guardrail-depth answer calls for directly here, even when no hook was added (e.g. "never run a destructive command without first explaining its blast radius").

## Pre-write checklist

- `name` passes the format rule above and has no unresolved collision.
- `description` is 1-3 sentences — long descriptions crowd automatic delegation matching.
- YAML is safe: quote any field value containing `:` or starting with a special character; no unescaped quotes inside plain scalars.
- Every field present matches what Phase 2's live research actually confirmed — never include a field this run didn't verify exists.
- The Phase 3 draft message lists every file the write step will create or modify by exact path (the agent file, any hook script, any companion skill's `SKILL.md` and `references/*.md`) — a plain change manifest, not just the file contents — immediately followed by an unmissable, visually distinct notice that nothing has been written yet (e.g. a horizontal rule and a bolded standalone line: **"Nothing has been created yet — confirm or adjust below."**). Do not fold that notice into the middle of prose; it must be its own visible block.

## Post-write checklist

- Re-read the file back; confirm the `---` frontmatter delimiters and every required field survived intact.
- If a `hooks:` block was written, confirm any referenced script file was actually created, and mention the workspace-trust caveat (`hooks.md`) in the summary.
- If a companion Skill was authored, confirm every `references/*.md` file it points to actually exists.
- Offer an optional smoke test: invoke the new subagent once via the Task tool on a trivial sample input, and confirm it loads and any hook fires without error.
- Summarize every choice made with its source, e.g. "tools: Read, Grep, Glob — from your answer", "model: sonnet — default", "maxTurns: 15 — default for open-ended tasks".
- Close with a **Managing this agent** note, since there's no enable/disable toggle for a standalone subagent (verify this is still accurate against the live schema check, same as every other claim here):
  - **Edit**: change the file directly; Claude Code picks up edits within seconds, no restart needed.
  - **Remove**: delete the file; that alone is sufficient, no cache to clear.
  - **Disable without deleting**: not natively supported per-agent. The user can block it themselves with `permissions.deny: ["Agent(<name>)"]` in their own `settings.json` — mention this as a manual option, never write `settings.json` for them (out of scope, see `SKILL.md`).
  - If this was the first agent file ever created in that level's `agents/` directory, note that Claude Code may need a restart to start watching the new directory.
