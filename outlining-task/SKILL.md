---
name: outlining-task
description: Use when starting any new feature work, before implementation. Covers both explicit requests ('outline this', 'define ACs', 'what are the acceptance criteria') and implicit 'build X' requests that need AC definition first.
---

# Outlining Task

Turn a feature idea into refined acceptance criteria plus an ordered sequence of
deliverables, through codebase exploration and structured grilling.

This skill decides **what ships and in what order**. `task-breakdown` then decides **how
each deliverable is built**.

## Process

### 1. Load the delivery vocabulary

Find the project's own words before writing a single AC. Search for a glossary or
ubiquitous-language doc — don't assume a fixed path, these move:

```bash
find . -name 'glossary*.md' -not -path '*/node_modules/*' -not -path '*/.git/*' -not -path '*/worktrees/*'
```

Common homes are `.agents/context/{feature}/`, `documentation/docs/features/{feature}/`
and `server/docs/`. If several turn up, take the one scoped to the feature in play.

- **Found one** — read it. Use its terms verbatim in every AC and deliverable name, and
  respect its _Avoid_ notes (e.g. say "workflow engine" not "legacy engine").
- **None** — ask the user what the canonical terms are for the domain nouns in play.

Never invent names, and never promote a code identifier into user-facing vocabulary.
`ReportingWorkflowExecutor` is not a word the reader of a ticket knows.

### 2. Explore existing work

Before drafting anything, explore the codebase to understand what already exists. Use
Explore agents to find:

- Existing implementations (APIs, handlers, services, components)
- Related types and interfaces
- Data model and relationships
- Feature flags or migration state
- Reference patterns from similar features already built

**Never draft ACs in the dark.** The codebase determines what's already done, what's
partially done, and what's missing.

### 3. Establish initial acceptance criteria

If the user provided ACs, use those as the starting point. If not, draft from codebase
exploration.

Each AC is **one user-visible capability, stated as an outcome**:

```
1. User can [action] [object]
```

State what becomes true, not how it is built. Implementation belongs in the ticket, not
the AC:

```
Good: User can filter widget data by field, with AND/OR
Bad:  User can filter widget data — mutually exclusive tabs send filter params to API
```

Where a criterion is not user-facing, state the observable behaviour instead. Include an
"Open Questions" section for anything ambiguous.

### 4. Grill to resolve ambiguity

Run a `/grilling` session on the ACs. Typical ambiguities to probe:

- **Scope boundaries** — What does "edit" mean? Full editor or just rename?
- **Missing user journeys** — How does the user get to this page? What happens after they submit?
- **Edge cases** — What if the list is empty? What if the user doesn't own the item?
- **API gaps** — Does the endpoint exist? Does it accept the params we need?
- **UX decisions** — Modal or page? Inline edit or dialog? Hidden or disabled for out-of-scope actions?
- **Domain model conflicts** — Do the filter categories overlap? Are they mutually exclusive?

### 5. Name and order the deliverables

An AC list says what will be true at the end. It does not say what ships, or when — and a
reader who cannot answer "what lands next?" will read the whole thing as an undifferentiated
pile of work. The deliverables sequence answers it.

**A deliverable is the smallest thing you could put in front of a user and have them
recognise it.** Name it after the surface they touch or the capability they gain:

```
Good: Datatable Tab, List View, Filter Tab, Bar Chart, Widget save state
Bad:  Schema layer, WorkflowExecutor, API wiring, Phase 2
```

Rules:

- **Flat names — no colons.** Bar, Line and Pie are three deliverables, not one
  `Visualization Tab: Bar Chart`. The name becomes a ticket title prefix downstream, where
  a colon inside it collides with the separator.
- **Order for delivery, not for build order.** Each deliverable should be usable on top of
  the ones before it. Persistence and cross-cutting guards go late — there is no point
  saving a configuration before there is a configuration worth saving.
- **Foundation work is not a deliverable.** Schemas, executors, restructures, planning and
  docs ship nothing a user sees. They stay out of the sequence and get no number. They
  still become tickets — see `task-breakdown`.

Confirm the sequence with the user. Ordering is a delivery decision, not a technical one.

### 6. Produce and save refined output

Three artifacts.

**Decision log** — every decision made during grilling:

```
| Decision | Answer |
|----------|--------|
| Detail page concept | New read-only view, not V1 Studio replacement |
| Edit scope | Rename only, full editor is future |
```

**Refined acceptance criteria** — outcome-framed, incorporating all decisions. Each
specific enough to implement without further clarification.

**Deliverables sequence** — numbered, in delivery order.

Together the last two are the parent story description:

```markdown
## Acceptance Criteria

1. User can select a data source for a workflow widget (project team + app)
2. User can add, remove and reorder data table columns
3. User can filter widget data (per-field-type operators, AND/OR)
4. User can save the widget and see the configuration restored when reopening

## Deliverables Sequence

1. Datatable Tab
2. List View
3. Filter Tab
4. Bar Chart
5. Line Chart
6. Pie Chart
7. Widget save state
```

Ask the user whether these should be saved to disk (e.g.
`documentation/docs/features/{feature-name}/`). Only save if they confirm.

Then hand off to `task-breakdown`, which slices each deliverable into tickets.
