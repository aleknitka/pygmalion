# Interview phase — question bank & rules

## Interaction rules

- Ask one question at a time; never stack sub-questions into one turn.
- Prefer single-select. Use multi-select only when the options genuinely coexist (e.g. tool access).
- Use the host's blocking-question tool when available (Claude Code's `AskUserQuestion`, capped at 4 explicit options); fall back to a plain numbered list in chat if no such tool exists.
- Use open-ended free text only when you genuinely can't write 3-4 distinct, non-padded options.
- Ask only decisions. Never ask something you can determine yourself by reading the repo or environment.

## Question order

**0. (Skip if already known)** If the task isn't already stated via `argument-hint` or the surrounding conversation, ask open-ended: *"What should this subagent do, and what should trigger Claude to use it?"*

**1. Agent level** (single-select) — determines both the write path and the Phase 0 collision check:
- Project (`.claude/agents/`) — checked into version control, shared with everyone working in this repo
- User (`~/.claude/agents/`) — personal, available in every project you open, not shared

**2. Tool access** (multi-select, framed toward least privilege). Offer candidates inferred from the stated task, for example:
- Read-only (`Read`, `Grep`, `Glob`)
- + editing (`Edit`, `Write`)
- + shell (`Bash`)
- + web (`WebSearch`, `WebFetch`)
- Specific MCP tools already configured in this project, if relevant

If `Bash` is selected, ask one optional free-text follow-up: *"Anything it should never be allowed to run?"* — this feeds `disallowedTools` and the hook `if` pattern later, if a hook is added.

**3. Model tier** (single-select): `haiku` (cheap, high-volume) / `sonnet` (default, balanced) / `opus` (complex reasoning) / `inherit` (match the session's model).

**4. Invocation style** (single-select): auto-delegated by Claude when relevant / name-only, manually invoked.

**5. Guardrail depth** (single-select):
- (a) Tool allowlist + instructions only — default
- (b) Add a scripted hook that blocks or validates specific actions
- (c) Add an LLM-judgment hook for cases too nuanced for a script
- (d) Not sure — recommend one after seeing the task

If the answer is (b), (c), or (d) resolving to (b)/(c), branch to `../hooks.md` during Phase 3 instead of asking further guardrail questions here.

**6. Companion skill** (single-select, conditional) — only ask this if Phase 2's domain research turns up substantial, stable reference material (a documented external methodology/spec/API) with no existing local Skill covering it. See `../skill-packaging.md` for the full decision procedure and package structure. Skip entirely, without asking, whenever no existing Skill matches *and* the domain doesn't warrant one — this is not a default question.

## What not to ask

- Whether the chosen name collides with an existing agent — checked via `Glob` in Phase 0/3, only surfaced as a question if a real collision is found.
- Whether a relevant Skill already exists to preload via `skills:` — checked in Phase 2's local-first scan, offered in the draft, never asked upfront. (Whether to *author a new one* is question 6 above, and only when warranted.)
