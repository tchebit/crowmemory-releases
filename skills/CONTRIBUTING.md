# Contributing Skills

CrowMemory skills are reusable prompt patterns that teach AI agents specialized
workflows using CrowMemory's memory tools.

## How to contribute

1. Fork this repository
2. Add your skill as `skills/<skill-name>/SKILL.md`
3. Open a PR

## Layout

A skill is a **folder** containing a `SKILL.md`, because that is the shape Claude
Code loads from `~/.claude/skills/` and `.claude/skills/`. Naming the folder after
the skill means a contributor can copy it straight in and it works:

```
skills/crow-memory-code-review/SKILL.md
skills/crow-memory-bug-triage/SKILL.md
skills/crow-memory-onboarding/SKILL.md
```

The folder may hold supporting files (reference docs, templates) alongside
`SKILL.md`; they travel with it when someone copies the directory.

## Frontmatter is required

`SKILL.md` must open with a YAML block:

```markdown
---
name: crow-memory-code-review
description: One or two sentences describing what the skill does AND when it applies.
---
```

- `name` must match the folder name.
- `description` is not decoration — the agent reads it to decide whether to load
  the skill at all. Write it for that job: say what the skill does *and* the
  situation that should trigger it. A vague description means the skill sits
  there unused.

## Body format

```markdown
# Human-Readable Title

> One-line summary.

## When to use

The scenario or trigger.

## Instructions

The workflow you are teaching. Reference CrowMemory tools by name
(remember, recall, link_memories, ...) — not by their full schema.

## Example

A short example showing the skill in action.
```

## Guidelines

- **One skill per folder** — keep each focused on a single workflow
- **No tool signatures** — reference tools by name, not by their full schema
- **No internals** — skills are public; describe behaviour, not implementation
- **No secrets or credentials**
- **Test it** — verify against at least one MCP client before submitting
- **Keep it concise** — a skill competes for context with the actual work

## Questions?

Open an [issue](https://github.com/tchebit/crowmemory-releases/issues) if you have
questions about the format or want feedback before submitting.
