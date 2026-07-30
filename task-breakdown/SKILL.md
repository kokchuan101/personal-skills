---
name: task-breakdown
description: Use when the user says 'break this into tickets', 'ticket structure', 'task breakdown', or has refined ACs ready to organize into implementable work units.
---

# Task Structure Breakdown

Break refined acceptance criteria into Jira tickets organized by user story.

## Ticket Tiers

Each user story is broken into sub-tickets by tier:

| Tier | Purpose | When to include |
|------|---------|-----------------|
| **[DB]** | Database migration | New tables, columns, or indexes needed |
| **[BE]** | Backend API | New endpoint or endpoint changes needed |
| **[FE]** | Frontend component | New UI component, modal, page, or skeleton |
| **[Integration]** | Connect FE to BE | Wire frontend actions to API calls, handle success/error flows |

**[DB] takes precedence over [BE]** — migrations must land before the API that depends on them.

## Ticket Description Structure

Every ticket follows this format:

```
**Context**
- What is missing or what problem this solves

**Acceptance Criteria**
- From user perspective: "User can [action]"
- Specific enough to verify when done
```

## Process

### 1. Group ACs by user story

Each acceptance criterion maps to a user story. One story = one atomic, tangible result from the user's perspective.

```
Story: User can create widget
Story: User can edit widget
Story: User can delete widget
Story: User can view widget list
Story: User can view widget detail page
```

### 2. Determine tiers per story

For each story, propose all applicable tiers:

- **Does this need a new table or column?** → [DB] ticket
- **Does this need a new or modified endpoint?** → [BE] ticket
- **Does this need new UI?** → [FE] ticket
- **Does this connect FE to BE?** → [Integration] ticket

Check the codebase to see what already exists. If the API endpoint is already built, the story may only need [FE] + [Integration].

Present all tiers and let the user decide what to merge or skip per-story.

### 3. Grill the breakdown

Run a `/grilling` session on the proposed breakdown. Question the structure:

- **Granularity** — Are tickets too coarse (multiple unrelated changes) or too fine (trivial standalone work)?
- **Dependencies** — Are cross-story dependencies clear? Does Story B's [FE] need Story A's [BE] to land first?
- **Missing stories** — Walk through user journeys. How does the user navigate between features? What happens on empty states, error states, permission boundaries?
- **Shared work** — If two stories need the same API hook or component, which story owns it?

### 4. Present and save the breakdown

List stories with their tier tickets. For each ticket, show Context and Acceptance Criteria.

Ask the user whether the ticket breakdown should be saved to disk (e.g. `documentation/docs/features/{feature-name}/`, alongside any existing ACs or decision logs for the feature). Only save if they confirm.
