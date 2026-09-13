# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project purpose

Agent Creator (packaged as the `pygmalion` plugin) is a Claude Code skill for creating subagents. The goal is to streamline building `.claude/agents/` subagent definitions for users by applying best practices and asking the right interview questions, producing agents that are robust, token-efficient, and able to act on the user's behalf.

The intended workflow for the agent-creation skill itself is:
1. Interview the user on the intended use of the agent.
2. Web search for best practices and patterns relevant to that use case.
3. Plan the agent, and any Skills/Tools it would need.
4. Consider whether hooks or guardrails should be part of the design.

## Architecture

This repo is packaged as a Claude Code **plugin** named `pygmalion` — there is no separate program to build or run. The manifest and marketplace catalog live at the repo root, and the implementation is the `agent-creator` skill it ships:

```
.claude-plugin/
  plugin.json                 # plugin manifest (name: pygmalion)
  marketplace.json            # single-plugin marketplace catalog (source: "./")
skills/agent-creator/
  SKILL.md                    # entry point: phase overview (interview -> research -> draft -> write)
  references/
    interview.md              # question bank + interaction rules for the interview phase
    output-spec.md            # full .claude/agents/*.md frontmatter contract, write location, field defaults, write checklist
    hooks.md                  # per-agent hooks.md: event constraints, hook-type selection, pattern library
    skill-packaging.md        # when/how to author a companion Skill alongside a created agent
```

Once installed elsewhere as a plugin, the skill is invoked as `/pygmalion:agent-creator` (namespaced by the plugin name). This repo's own `.claude/agents/`, if populated, holds project-specific subagents for working on this repo itself — unrelated to the plugin's own shipped components.

The skill writes agents at whichever level (project `.claude/agents/` or user `~/.claude/agents/`) the interview settles on — never assume project-level. It can also author a companion Skill alongside an agent (see `skill-packaging.md`) rather than inlining large reference material into the system prompt; only do this when the domain genuinely has enough distinct sub-topics to benefit from progressive disclosure, matched to the same level as the agent. There is no native per-agent enable/disable toggle in Claude Code — the write-up must tell users to edit or delete the file directly, and that disabling without deleting requires a manual `permissions.deny` entry in their own `settings.json` (never written by this skill, per the existing settings.json boundary below).

`SKILL.md` stays deliberately lean (Claude Code auto-compacts each skill body to ~5,000 tokens, with a 25,000-token combined budget across all active skills in a session) and links out to the `references/*.md` files for phase-specific detail — read those before editing the corresponding phase.

The skill's own "write the agent file" step **re-verifies the current `.claude/agents/*.md` schema via WebSearch/WebFetch on every run** rather than trusting a hardcoded field list — the schema is versioned and richer than a minimal `name/description/tools/model` sketch (it also includes fields like `disallowedTools`, `permissionMode`, `maxTurns`, `skills`, `memory`, `isolation`, and a per-agent `hooks:` block). Keep this "verify live, don't hardcode" behavior when editing `references/output-spec.md` or `references/hooks.md`.

Per-agent `hooks:` frontmatter only supports three lifecycle events — `PreToolUse`, `PostToolUse`, `Stop` — not the full Claude Code hook catalog. See `references/hooks.md` for the full design (hook type selection, script-invocation convention, reusable pattern library).

v1 targets Claude Code only — see Roadmap below for what comes after.

## Roadmap

Planned, in this order — none of it exists yet, and none of it should be built preemptively:

1. **v2 — other agentic-coding harnesses**: extend output support to Codex, Pi, and OpenCode, each with their own subagent/config format and frontmatter schema (to be researched live the same way the Claude Code schema is, not assumed from this repo's current knowledge). Keep the interview/research logic conceptually separable from the Claude-Code-specific output template where that separation is free, without building out a multi-harness abstraction now — a harness-selection question in the interview only makes sense once a second output template actually exists.
2. **v3 — Python CLI agents**: after the agentic-coding harnesses, extend to generating standalone Python CLI agents (not tied to any of the above harnesses) — same interview-driven approach, different output shape entirely (a runnable script/package rather than a markdown+frontmatter file).

## Current state

No test suite, linter, or build tooling applies here — this is a Markdown-only skill definition, verified by using it end-to-end in a live Claude Code session (invoke `/agent-creator` or, once installed as a plugin, `/pygmalion:agent-creator`) rather than automated tests.
