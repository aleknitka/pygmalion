---
name: agent-creator
description: "Interview the user, research best practices, then generate a Claude Code subagent file at .claude/agents/<name>.md with correct frontmatter (name, description, tools, model, optional guardrails/hooks) and a token-efficient system prompt. Use when the user wants to create, define, or scaffold a new persistent Claude Code subagent, or says things like 'make me an agent that does X' meaning a reusable .claude/agents/ file invoked via the Task tool. Not for one-off in-session delegation with no persistent file, not for authoring Claude Code Skills (SKILL.md) or slash commands, and not for agents targeting other harnesses (Codex, OpenCode, Pi) — v1 is Claude Code subagents only."
argument-hint: "[what the new subagent should do]"
allowed-tools: AskUserQuestion, WebSearch, WebFetch, Read, Glob, Grep, Write, Edit
---

# Agent Creator

Done when a `<agents-dir>/<name>.md` file exists — at whichever level (project `.claude/agents/` or user `~/.claude/agents/`) the interview settled on — with valid, currently-correct frontmatter and a coherent, token-efficient system prompt confirmed by the user, plus any companion Skill it needed — or the user stops the flow first, in which case nothing is written.

| Phase | Read first |
|---|---|
| 1 Interview | `references/interview.md` |
| 2 Research | `references/skill-packaging.md` if a companion Skill looks warranted |
| 3 Draft | `references/output-spec.md` (+ `references/hooks.md` if a hook is warranted, `references/skill-packaging.md` if a companion Skill is being authored) |
| 4 Write | `references/output-spec.md` |

## Phase 0 — Update or create?

If the request names an existing agent, or `Glob .claude/agents/*.md` and `~/.claude/agents/*.md` turns up a name match for the stated task, `Read` that file and ask whether this is a revision or a new agent alongside it.
- **Revision**: skip straight to a targeted version of Phase 1 ("what should change?") then Phase 3 using `Edit`, not a full redraft.
- **New**: continue to Phase 1.

## Phase 1 — Interview

Read `references/interview.md` before asking anything, and follow its interaction rules exactly (one question at a time, ≤4 options, ask only real decisions). This is also where the agent's level (project vs. user) gets decided — it determines the write path for every later phase.

## Phase 2 — Research

1. **Local-first**: `Glob`/`Grep` over `.claude/skills/**/SKILL.md`, `.claude/agents/*.md`, `~/.claude/agents/*.md`, and `~/.claude/skills/**/SKILL.md` — both for existing conventions to stay consistent with, and for Skills whose content could be preloaded into the new agent via the `skills:` frontmatter field instead of duplicating that knowledge in its system prompt. If nothing matches and the domain has enough distinct reference material to warrant its own package, read `references/skill-packaging.md` and ask the interview's companion-skill question before moving on.
2. **Domain research**: `WebSearch` for best practices specific to the task the new agent will perform. If authoring a companion Skill, this is also where its reference content gets sourced — from the actual search/fetch results, not memory.
3. **Schema research (mandatory every run)**: `WebSearch`/`WebFetch` the *current* `.claude/agents/*.md` frontmatter schema against official Claude Code docs. Never write a field from memory — the schema is versioned and richer than a minimal `name/description/tools/model` set (it also includes things like `disallowedTools`, `permissionMode`, `maxTurns`, `skills`, `memory`, `isolation`, `hooks`, and more); confirm the current field list and valid values live before drafting.

## Phase 3 — Draft

Read `references/output-spec.md`. Assemble the frontmatter and system prompt from the interview + research, writing to the level decided in Phase 1. If the guardrail-depth answer calls for a hook, also read `references/hooks.md` and assemble the `hooks:` block there; if a companion Skill was warranted, draft it per `references/skill-packaging.md` alongside the agent. Present the full draft in chat, per `output-spec.md`'s pre-write checklist: an explicit file-by-file change manifest followed by its own unmissable, visually distinct notice that nothing has been written yet — then get one confirm/adjust answer before writing anything.

## Phase 4 — Write, validate, summarize

Follow the pre-write and post-write checklists in `references/output-spec.md`. Create the target `agents/` directory (and the companion Skill's directory, if any) if absent, write every file from the confirmed manifest, re-read each to confirm integrity, optionally smoke-test by invoking the new subagent once via the Task tool on a trivial input, then summarize every choice made with its source (e.g. "tools: from your answer", "model: sonnet, default") and close with the Managing-this-agent note from `output-spec.md` (edit/remove/disable, restart caveat).

Never write to `.claude/settings.json` or attempt project-wide hook setup — that's out of scope; point the user at a manual follow-up instead if that's really what they need.
