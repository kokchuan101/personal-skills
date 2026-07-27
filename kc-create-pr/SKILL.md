---
name: create-pr
description: Use when implementation is done and the user wants to ship — create branch, commit, push, and open a PR. Requires a Jira ticket in conversation context. Triggers on "ship it", "create a PR", "push and open PR", "create-pr".
---

# Create PR

Create a branch, commit, push, and open a PR from the current working state. Requires a Jira ticket number in conversation context.

## Prerequisites

A Jira ticket must be present in the conversation. If not, ask the user for one — do not proceed without it.

## Step 1 — Gather Context

Run in parallel:

```bash
git status
git diff --stat
git log --oneline -5
```

Determine what work remains: uncommitted changes, unpushed commits, or everything already pushed.

## Step 2 — Determine Branch Type

Fetch the Jira issue type to choose the branch prefix:

| Issue Type | Prefix |
|---|---|
| Story, Sub-task | `feature/` |
| Bug | `bugfix/` |
| Task | `task/` |

Use `getJiraIssue` to fetch the issue type. If the Atlassian MCP tool is unavailable, ask the user: "Is this a feature, bugfix, or task?"

## Step 3 — Create Branch

Check the current branch first:

- **Already on a matching branch** (e.g. `feature/DEV-21405`) — skip, stay on it.
- **On any other branch** — always branch from latest `main`:

```bash
git fetch origin main
git checkout -b {prefix}/{JIRA-ticket} origin/main
```

**Never branch from the current working branch.** Always use `origin/main` as the base to avoid carrying unrelated commits into the PR.

## Step 4 — Stage and Commit

If there are no uncommitted changes, skip this step.

**Staging:** Add specific files by name. Never use `git add -A` or `git add .`.

**Commit message format:**

```bash
git commit -m "$(cat <<'EOF'
{JIRA-ticket}: Brief imperative description

Why this change was made — not how. Wrap at ~72 chars.

Co-Authored-By: Claude Opus 4.6 (1M context) <noreply@anthropic.com>
EOF
)"
```

Rules:
- Subject: `{JIRA-ticket}: description` — max 72 chars, imperative mood, no trailing period
- Body: explains *why*, not *what* (the diff shows what)
- Do not squash or rebase existing commits on the branch

## Step 5 — Push

If the branch has no unpushed commits, skip.

```bash
git push -u origin {branch}
```

## Step 6 — Create PR

Read the PR template if one exists:

```bash
cat .github/pull_request_template.md
```

Create the PR targeting `main`:

```bash
gh pr create --base main --title "{JIRA-ticket}: Brief title" --body "$(cat <<'EOF'
{PR body here}
EOF
)"
```

**Title:** `{JIRA-ticket}: brief title` — under 70 chars.

**Body:** Synthesize from the conversation context, the diff, and the Jira ticket description. Be concise and straight to the point:
- **Why** — the problem or need (1-2 sentences)
- **How** — the approach taken (bullets)
- **Side effects** — breaking changes or behavior shifts. Leave blank if none.
- **Testing** — how it was verified (test commands, manual steps)
- **Release notes** — leave blank unless user-facing

For checklist items, check the ones that genuinely apply. Leave unchecked items that weren't addressed.

Report the PR URL when done.
