# Contributing Skills

CrowMemory skills are reusable prompt patterns that teach AI agents specialized workflows using CrowMemory's memory tools.

## How to contribute

1. Fork this repository
2. Create your skill file in the `skills/` folder
3. Follow the naming convention: `crow-memory-skill-<variant-name>.md`
4. Open a PR

## File naming

```
skills/crow-memory-skill-code-review.md
skills/crow-memory-skill-bug-triage.md
skills/crow-memory-skill-onboarding.md
skills/crow-memory-skill-architecture-log.md
```

## Skill file format

Every skill file must follow this structure:

```markdown
# crow-memory-skill-<variant-name>

> One-line description of what this skill does.

## When to use

Describe the scenario or trigger for this skill.

## Instructions

The prompt instructions that teach the AI agent the workflow.
Reference CrowMemory tools by name (remember, recall, hybrid_recall, etc.)
but do not include tool signatures or implementation details.

## Example

A short example showing the skill in action.
```

## Guidelines

- **One skill per file** — keep each skill focused on a single workflow
- **No tool signatures** — reference tools by name, not by their full schema
- **No secrets or credentials** — skills are public
- **Test your skill** — verify it works with at least one MCP client before submitting
- **Keep it concise** — skills should be easy to scan and understand

## Questions?

Open an [issue](https://github.com/tchebit/crowmemory-releases/issues) if you have questions about the format or want feedback before submitting.
