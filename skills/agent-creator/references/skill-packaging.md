# Companion skill packaging — when and how to author a new Skill alongside an agent

Read this only when Phase 2's local-first scan finds no existing Skill covering the domain, *and* the domain research surfaced enough reference material that inlining it would bloat the system prompt (a documented external methodology, spec, or API with several distinct sub-topics — not a two-paragraph summary).

## Decide, don't default

Ask the interview's companion-skill question (see `interview.md` question 6) with these options, sized to the actual material found:
- Author a new Skill — recommended when the material has natural sub-sections (e.g. four distinct categories, a multi-endpoint API) that benefit from progressive disclosure
- Inline in the system prompt — fine for a compact, single-shot set of rules that won't grow
- Not sure — recommend based on the size/structure of what Phase 2 actually found

Never author a Skill for material that's small enough to state once in the agent's system prompt — that's needless indirection for no benefit.

## Structure — mirror progressive disclosure

```
.claude/skills/<skill-name>/
  SKILL.md              # lean entry point: overview + a table pointing into references/
  references/
    <topic-a>.md         # loaded on demand, one file per natural sub-topic
    <topic-b>.md
```

- `SKILL.md` frontmatter needs `name` and a `description` written as "Use when X" (same pattern as agent `description`) so it can also be discovered/matched independently of this one agent.
- Keep `SKILL.md` itself short (it may load in sessions where the full detail isn't needed) — one line per sub-topic plus a pointer to its reference file, not the content itself.
- Source reference-file content from the actual domain research (WebFetch/WebSearch results), not from memory — same "verify live" rule as the agent schema itself.

## Wire it into the agent

- Add the skill's name to the agent's `skills:` frontmatter list so its full content is preloaded at the agent's startup.
- Do **not** duplicate the skill's content in the agent's system prompt — the system prompt should state the working rules specific to *this agent's* task (what to do with the knowledge) and defer the domain knowledge itself to the skill.

## Where it's created

Match the skill's location to the agent's chosen level from the interview: a project-level agent gets a project-level companion skill (`.claude/skills/`), a user-level agent gets a user-level one (`~/.claude/skills/`). Mention this pairing explicitly in the Phase 3 draft and the Phase 4 summary — it's a second artifact being created, not an implicit side effect.
