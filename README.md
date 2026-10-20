# personal-skills

Personal [Agent Skills](https://github.com/vercel-labs/skills) I use with Claude Code.

| Skill | What it does |
|---|---|
| `create-pr` | Ships finished work — branches from `origin/main`, commits, pushes, and opens a PR. Expects a Jira ticket in conversation context. |
| `outlining-task` | Turns a feature idea into outcome-framed acceptance criteria plus an ordered sequence of named deliverables, before implementation starts. |
| `task-breakdown` | Slices a deliverables sequence into Jira tickets — vertical or horizontal, each prefixed with the deliverable it ships and grouped in delivery order. |
| `workflow-tlr-parity` | Verifies and pins behavior parity when re-implementing a legacy Checkbox feature in the TLR widget studio — parity matrix, divergence record, matrix-grouped tests, three-tier verification loop. Only works inside the `cbdev` repo. |

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

No CLI needed — copy the skill directory across:

```bash
cp -R create-pr ~/.claude/skills/
```

Most skills are a single self-contained `SKILL.md`; `workflow-tlr-parity` also
ships `references/`, `assets/`, and `evals/` alongside it, so copy the whole
directory.

## Adding a skill here

```bash
npx skills init my-new-skill
```

Then fill in the `name` and `description` frontmatter in `my-new-skill/SKILL.md`. The
`description` is what the agent matches against, so lead with the trigger conditions.
