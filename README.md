<div align="center">
  <img src="media/pygmalion_logo.png" alt="Pygmalion logo" width="200">
</div>

# Agent Creator

**Why "Pygmalion"?** In Greek myth, the sculptor Pygmalion carved a statue so lifelike that it was brought to life. That's the shape of what this skill does: you describe what you want, it crafts a `.claude/agents/*.md` definition from that description, and the result is something that then acts on its own — a subagent, not a static artifact.

<div align="center">
  <a href="https://code.claude.com/docs/en/claude-code"><img src="https://img.shields.io/badge/Claude_Code-555?logo=claude" alt="Claude Code"></a>
</div>

A Claude Code Skill that streamlines creating [Claude Code subagents](https://code.claude.com/docs/en/sub-agents). Instead of hand-writing a `.claude/agents/<name>.md` file and guessing at frontmatter, it interviews you about what the subagent should do, researches current best practices and the live subagent schema, and writes a robust, token-efficient agent file at whichever level you choose — hooks, guardrails, and a companion Skill included, if you want them.

> v1 targets Claude Code only. Planned next: support for other agentic-coding harnesses (Codex, OpenCode, Pi) with the same streamlined flow, then standalone Python CLI agents — see `CLAUDE.md`'s Roadmap.

## What it does

Invoking the skill runs a short, four-phase flow:

1. **Interview** — one question at a time: what should it do, project-level or user-level, what tools should it touch, which model tier, how it should be invoked, how strict its guardrails should be, and — only when the domain genuinely calls for it — whether to package its reference material as a companion Skill instead of inlining it.
2. **Research** — checks your project and home directory for existing Skills/agents worth reusing, looks up best practices for the specific task, and re-verifies the *current* `.claude/agents/*.md` frontmatter schema live (it doesn't trust a hardcoded field list, since the schema is versioned).
3. **Draft** — assembles the frontmatter and system prompt, including any hooks the guardrail answer calls for and any companion Skill, and shows you a file-by-file change manifest with an unmissable notice that nothing has been written yet, before you confirm.
4. **Write** — creates the agent file (and companion Skill, if any) at the chosen level, re-reads it to confirm it's well-formed, optionally runs a smoke test, summarizes every choice it made and why, and closes with how to manage the new agent going forward (edit, remove, or disable it).

It never touches `.claude/settings.json` or sets up project-wide hooks — guardrails it adds are always scoped to the one agent file it owns. Disabling an agent without deleting it isn't natively supported by Claude Code, so that step is a manual `settings.json` change you make yourself; the write-up tells you exactly what to add.

## Installation

This repo is packaged as a Claude Code **plugin** named `pygmalion`, distributed through its own single-plugin marketplace catalog. From any Claude Code session:

```
/plugin marketplace add aleknitka/pygmalion
/plugin install pygmalion@pygmalion
```

Once installed, the skill runs as `/pygmalion:agent-creator` (namespaced by the plugin name).

### Alternative: skills.sh

The repo's `skills/agent-creator/SKILL.md` layout is also directly installable via [skills.sh](https://skills.sh)'s CLI, no separate submission needed:

```
npx skills add aleknitka/pygmalion
```

### Alternative: unpackaged, for local hacking on the skill itself

Claude Code also discovers skills directly from a **project's** `.claude/skills/`, or your **user-wide** `~/.claude/skills/`, without any plugin/marketplace step — useful if you're editing this skill's own source. Clone this repo, then either symlink or copy `skills/agent-creator/` from it into one of those locations. Run unpackaged this way, it's invoked unnamespaced as `/agent-creator`.

## Usage

Inside a Claude Code session, invoke it directly:

```
/pygmalion:agent-creator a subagent that reviews incoming GitHub issues and labels them
```

or just describe what you want in conversation — the skill's description is written to auto-trigger on requests like "make me an agent that does X." Either way, it will ask a few questions before writing anything, and always shows you the draft first.

## Updating

```
/plugin marketplace update pygmalion
/plugin update pygmalion
```

## Repository layout

```
.claude-plugin/
  plugin.json              # plugin manifest (name: pygmalion)
  marketplace.json         # single-plugin marketplace catalog (source: "./")
skills/agent-creator/
  SKILL.md                # entry point: the four-phase flow
  references/
    interview.md          # interview question bank + interaction rules
    output-spec.md         # .claude/agents/*.md frontmatter contract, write location & checklist
    hooks.md               # per-agent hook design: event constraints, hook types, pattern library
    skill-packaging.md     # when/how to author a companion Skill alongside an agent
CLAUDE.md                  # guidance for Claude Code instances working on this repo, incl. roadmap
```

## License

No license file yet — treat as all-rights-reserved until one is added.
