# personal-skills

Personal [Agent Skills](https://github.com/vercel-labs/skills) I use with Claude Code.

| Skill | What it does |
|---|---|
| `create-pr` | Ships finished work — branches from `origin/main`, commits, pushes, and opens a PR. Expects a Jira ticket in conversation context. |
| `tlr-widget-studio-context` | Loads TLR widget studio docs (glossary, spec, and task-relevant references) at the start of a reporting-widget session. Only works inside the `cbdev` repo. |

## Install

Install everything, globally (available in every project):

```bash
npx skills add kokchuan101/personal-skills --global
```

Install into the current project only (writes to `./.claude/skills/`):

```bash
npx skills add kokchuan101/personal-skills
```

Pick a single skill:

```bash
npx skills add kokchuan101/personal-skills --skill create-pr --global
```

Useful flags:

- `-l, --list` — show what's in the repo without installing
- `-a, --agent <agents>` — target specific agents (`'*'` for all); defaults to prompting
- `--copy` — copy files instead of symlinking
- `-y, --yes` — skip confirmation prompts

## Manage

```bash
npx skills list                 # what's installed
npx skills update               # pull latest versions
npx skills remove create-pr     # uninstall
```

## Manual install

No CLI needed — each skill is a single self-contained `SKILL.md`:

```bash
cp -R create-pr ~/.claude/skills/
```

## Adding a skill here

```bash
npx skills init my-new-skill
```

Then fill in the `name` and `description` frontmatter in `my-new-skill/SKILL.md`. The
`description` is what the agent matches against, so lead with the trigger conditions.
