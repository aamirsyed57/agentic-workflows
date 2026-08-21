# agentic-workflows

Agentic workflows and skills for LLM brains for software development.

## Layout

Each skill lives under `skills/<name>/`:

```
skills/
└── audit/
    ├── SKILL.md               # skill source: YAML frontmatter + instructions
    └── codebase-audit.skill   # the same skill packaged as a zip
```

- `SKILL.md` is what Claude Code reads directly from a skills directory.
- `<name>.skill` is a zip of the skill folder (`SKILL.md` + any `references/`), for surfaces that take a packaged upload instead of a directory — claude.ai, Claude Desktop, and the Skills API.

Current skills:

| Skill | Path | What it does |
|---|---|---|
| `codebase-audit` | `skills/audit/` | Audits an app/codebase across architecture, code quality, dependencies, and recurring cost, then reports what to cut or simplify. |

## Adding a skill to another project (Claude Code)

Claude Code loads skills from two places:

| Scope | Path | Applies to |
|---|---|---|
| Personal | `~/.claude/skills/<skill-name>/SKILL.md` | every project you open |
| Project | `.claude/skills/<skill-name>/SKILL.md` | that project only (checked into the repo, shared with the team) |

To add the audit skill to another repo, copy the folder in:

```bash
# project-scoped (recommended for team-shared skills — commit .claude/skills/)
cp -r skills/audit path/to/other-repo/.claude/skills/audit

# or personal, available everywhere
cp -r skills/audit ~/.claude/skills/audit
```

To keep it updatable instead of a one-time copy, symlink it from a clone of this repo — Claude Code follows symlinks in the skills directories:

```bash
git clone https://github.com/<you>/agentic-workflows ~/src/agentic-workflows
ln -s ~/src/agentic-workflows/skills/audit ~/.claude/skills/audit
```

Notes:
- The command you type (`/audit`) comes from the **directory name**, not the `name:` field in the frontmatter — rename the destination folder if you want a different command name.
- Claude Code picks up additions/edits under `~/.claude/skills/` and a project's `.claude/skills/` live, without a restart, as long as the top-level skills directory already existed when the session started.
- If a skill of the same name exists at more than one scope, enterprise beats personal, and personal beats project.

## Adding a skill to claude.ai / Claude Desktop / the API

These surfaces take the packaged `.skill` zip instead of a raw folder:

1. Take the file, e.g. `skills/audit/codebase-audit.skill`.
2. claude.ai or Claude Desktop: Settings → Capabilities → Skills → upload the file.
3. API: upload it through the Skills API.

Note the packaged form only supports the subset of frontmatter fields in the open Agent Skills spec (`name`, `description`, `license`, `compatibility`, `metadata`, `allowed-tools`) — Claude Code-only fields like `disable-model-invocation` or `context: fork` are dropped if present.

## Adding a new skill to this repo

1. Create `skills/<name>/SKILL.md` with a `description` (and optionally `name`) in the frontmatter, plus any supporting files (e.g. `references/*.md`) alongside it.
2. Test it locally by copying/symlinking into `.claude/skills/` or `~/.claude/skills/` as above.
3. If you also want it distributable to claude.ai/Desktop/the API, package the folder into `<name>.skill` (a zip with `SKILL.md` at its root) and commit both.

## License

GPLv3 — see [LICENSE](LICENSE).
