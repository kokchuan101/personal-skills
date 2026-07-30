---
name: outlining-task
description: Use when starting any new feature work, before implementation. Covers both explicit requests ('outline this', 'define ACs', 'what are the acceptance criteria') and implicit 'build X' requests that need AC definition first.
---

# Outlining Task

Turn a feature idea into refined acceptance criteria through codebase exploration and structured grilling.

## Process

### 1. Explore existing work

Before drafting anything, explore the codebase to understand what already exists. Use Explore agents to find:

- Existing implementations (APIs, handlers, services, components)
- Related types and interfaces
- Data model and relationships
- Feature flags or migration state
- Reference patterns from similar features already built

**Never draft ACs in the dark.** The codebase determines what's already done, what's partially done, and what's missing.

### 2. Establish initial acceptance criteria

If the user provided ACs, use those as the starting point. If not, draft from codebase exploration.

Format ACs from the user's perspective:

```
1. User can [action] [object]
```

Each AC is a single user-visible capability. Include an "Open Questions" section for anything ambiguous.

### 3. Grill to resolve ambiguity

Run a `/grilling` session on the ACs. Typical ambiguities to probe:

- **Scope boundaries** — What does "edit" mean? Full editor or just rename?
- **Missing user journeys** — How does the user get to this page? What happens after they submit?
- **Edge cases** — What if the list is empty? What if the user doesn't own the item?
- **API gaps** — Does the endpoint exist? Does it accept the params we need?
- **UX decisions** — Modal or page? Inline edit or dialog? Hidden or disabled for out-of-scope actions?
- **Domain model conflicts** — Do the filter categories overlap? Are they mutually exclusive?

### 4. Produce and save refined output

After grilling, produce:

**Decision log** — Table of every decision made during grilling:

```
| Decision | Answer |
|----------|--------|
| Detail page concept | New read-only view, not V1 Studio replacement |
| Edit scope | Rename only, full editor is future |
```

**Refined acceptance criteria** — Updated ACs incorporating all decisions:

```
1. User can view widget list — fetches from V2 API with pagination
2. User can filter widget list — mutually exclusive tabs send filter params to API
```

Each AC should be specific enough to implement without further clarification.

Ask the user whether the decision log and refined ACs should be saved to disk (e.g. `documentation/docs/features/{feature-name}/`). Only save if they confirm.
