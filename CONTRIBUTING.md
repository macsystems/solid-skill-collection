# Contributing

Thanks for adding to the collection! Every skill is a folder under `skills/` containing a
`SKILL.md` file. This document explains the format and the checklist for a new skill.

## Anatomy of a skill

```
skills/
  <skill-name>/
    SKILL.md          # required
    reference.md      # optional — extra docs the agent can open on demand
    scripts/          # optional — helper scripts the skill may run
    examples/         # optional — before/after samples
```

- `<skill-name>` must be lowercase, using hyphens (e.g. `single-responsibility`).
- The folder name should match the `name` in the frontmatter.

## `SKILL.md` format

A `SKILL.md` is Markdown with a YAML frontmatter block at the top:

```markdown
---
name: single-responsibility
description: Use when reviewing or writing a class/module to keep it focused on one
  reason to change. Flags God objects, mixed concerns, and over-broad interfaces, and
  suggests how to split responsibilities.
---

# Single Responsibility Principle

Instructions for the agent go here...
```

### Frontmatter fields

| Field | Required | Notes |
| ----- | -------- | ----- |
| `name` | yes | Lowercase, hyphenated. Matches the folder name. |
| `description` | yes | The most important field. Written in the third person, it tells the agent **when** to use the skill. Lead with the trigger ("Use when..."). |
| `license` | no | Defaults to the repo license (Apache-2.0). |
| `allowed-tools` | no | Restrict which tools the skill may use, if applicable. |

### Writing a good `description`

The agent decides whether to load a skill based solely on its `description`, so make it
about triggers and outcomes, not implementation. Mention the symptoms or requests that
should activate it.

### Writing the body

- Keep it focused and actionable — the body is instructions to the agent, not an essay.
- Prefer concrete heuristics, checklists, and before/after examples.
- Move long references into separate files in the skill folder and point to them, so the
  main `SKILL.md` stays lean.

## Checklist for a new skill

- [ ] Folder under `skills/` named after the skill.
- [ ] `SKILL.md` with valid `name` + `description` frontmatter.
- [ ] `name` matches the folder name.
- [ ] Added a row to the **Available skills** table in `README.md`.
- [ ] Tested by installing it locally (see README) and triggering it.

Start from [`templates/SKILL.md`](templates/SKILL.md).
