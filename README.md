# solid-skill-collection

A collection of [Agent Skills](https://docs.claude.com/en/docs/claude-code/skills) that help coding agents apply and enforce the **SOLID** principles of object-oriented design.

Each skill is a self-contained folder with a `SKILL.md` file. The format follows the open Agent Skills convention, so the skills work in **Claude Code**, the **Claude apps**, and any other agent that understands `SKILL.md`.

## Available skills

<!-- Keep this table in sync when adding a skill. -->

| Skill | What it does |
| ----- | ------------ |
| [`dependency-inversion-principle`](skills/dependency-inversion-principle/) | Split a component into a stable public `api` module and a hidden `impl` module so clients depend only on the contract — covers factory/builder/DI entry points and module-coupling rules. |

## Installation

Skills are plain folders — pick whichever scope fits.

### Per project (recommended for teams)

Copy the skill you want into your project's `.claude/skills/` directory:

```bash
# from the root of your project
mkdir -p .claude/skills
cp -r /path/to/solid-skill-collection/skills/<skill-name> .claude/skills/
```

Committing `.claude/skills/` shares the skill with everyone on the repo.

### For your user account (available in every project)

```bash
mkdir -p ~/.claude/skills
cp -r /path/to/solid-skill-collection/skills/<skill-name> ~/.claude/skills/
```

### Try the whole collection quickly

```bash
git clone https://github.com/<your-org>/solid-skill-collection.git
ln -s "$(pwd)/solid-skill-collection/skills/<skill-name>" ~/.claude/skills/<skill-name>
```

## Using a skill

Once installed, the agent picks a skill up automatically when your request matches its
`description`. You can also invoke one explicitly in Claude Code with `/<skill-name>`.

## Repository layout

```
skills/
  <skill-name>/
    SKILL.md          # required — frontmatter + instructions
    ...               # optional supporting files (scripts, references, templates)
```

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for the `SKILL.md` format and how to add a new
skill. A ready-to-copy starting point lives in [`templates/SKILL.md`](templates/SKILL.md).

## License

[Apache License 2.0](LICENSE).
