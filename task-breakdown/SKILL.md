---
name: task-breakdown
description: Use when the user says 'break this into tickets', 'ticket structure', 'task breakdown', or has refined ACs ready to organize into implementable work units.
---

# Task Structure Breakdown

Break refined acceptance criteria into Jira tickets, each grouped under the deliverable it
ships and ordered by the delivery sequence.

`outlining-task` decides what ships and in what order. This skill decides **how each
deliverable is built**.

## Input

Two things are needed:

1. **Refined acceptance criteria** — outcome-framed, one user-visible capability each
2. **A deliverables sequence** — named, ordered, flat (no colons)

If there is no sequence, produce one first (see `outlining-task` step 5). **Never slice
against an unordered AC list** — the output is a pile of work items where nobody can tell
what lands next, which is the exact failure this structure exists to prevent.

Load the delivery vocabulary as `outlining-task` step 1 describes — read the project
glossary if one exists, otherwise ask the user. Ticket titles get read by people who
weren't at the grooming.

## Step 1 — Ask the slicing axis

Ask before proposing any tickets. Do not assume.

- **Vertical** — one ticket per deliverable, end to end (schema → API → UI → wired).
  Demoable on merge. Best when one person carries the whole deliverable.
- **Horizontal** — separate `[DB]` / `[BE]` / `[FE]` / `[Integration]` tickets per
  deliverable. Best when disciplines split across people, or when the UI can proceed
  against fixtures while the API is still settling.

Either way the deliverable prefix and the sequence order stay. Only granularity changes.

## Step 2 — Slice each deliverable

Work through the deliverables in sequence order.

### Vertical

One ticket per deliverable. If a deliverable is too big for one ticket, split it by
**capability** — never by layer, or you have silently chosen horizontal.

### Horizontal

| Tier | Ships |
|---|---|
| **[DB]** | Migration — new tables, columns, indexes |
| **[BE]** | An endpoint, or a change to one |
| **[FE]** | UI — component, modal, page, skeleton |
| **[Integration]** | **The gaps between the other tiers** — wiring the UI to the API, plus the loading, empty, error and success states that only exist once the parts meet |

`[Integration]` is what turns parts into a deliverable. It is normally the last ticket of a
deliverable and the one that makes it demoable, so every deliverable needs one unless a
single ticket already covers the whole path.

`[DB]` precedes `[BE]` — a migration must land before the API that depends on it.

Compound tiers like `[FE+Integration]` are fine when one ticket genuinely spans both.

Check the codebase for what already exists. If the endpoint is already built, a deliverable
may only need `[FE]` + `[Integration]`.

Present the proposed tiers per deliverable and let the user merge or skip.

## Step 3 — Title every ticket

```
[<Tier>] <Deliverable>: <verb> <object>     horizontal
<Deliverable>: <verb> <object>              vertical — no single discipline to name
```

- `<Deliverable>` is copied **verbatim** from the deliverables sequence, so every ticket
  belonging to one deliverable shares a prefix and the board groups itself.
- **Omit the deliverable when the ticket ships nothing user-visible** — foundation,
  restructure, planning, docs, fixture-only UI. Prefix absence is meaningful: it says "this
  is not part of a named deliverable."
- **One verb, one object.** No class names, file paths, library names, or `+`-joined
  objects. (This constrains the object, not the tier prefix.)
- Prefer `Add` / `Extract` / `Split` / `Connect` / `Show` over `Build` / `Lift` / `Expose`
  / `Render` / `Wire`.
- Single spaces only.
- Terminal states go in the title: `— DROPPED`, `(resolved by reuse)`.

De-jargoning is most of the work:

| Don't | Do |
|---|---|
| `[BE] Add ReportingWorkflowExecutor + Go client and register the WORKFLOW factory case` | `[BE] Add workflow executor and wire it into the factory` |
| `[FE] Lift v1 datagrid + cell formatters into reporting/shared/datagrid` | `[FE] List View: Extract shared datagrid and cell formatters from v1` |

## Step 4 — Write each description

```markdown
## Context
- <only the why that constrains an AC>
- Depends on: <ticket keys>

## Acceptance Criteria
1. User can <capability>
```

`Depends on:` is always the last Context bullet — that's where the reader looks for it.

The rules that matter, because this is where breakdowns usually fail:

- **If a Context bullet does not constrain an acceptance criterion, it does not belong in
  the ticket.** Link it or delete it.
- **Context must not be longer than the AC list.** If it is, cut Context.
- **Cap Context at ~4 bullets.** Endpoint inventories, `file:line` references and
  hook-by-hook implementation specs get linked out, never inlined.
- Group ACs under `###` sub-headings once past ~6.
- **Delete superseded content.** Never retain it under a "superseded" heading.
- When a ticket outgrows itself, **split it rather than grow its AC list**, and leave
  provenance in both halves (`Split out of CB-510, which covers the renderer half`).

The compression this asks for, on a real ticket:

**Before** — ~1,900 chars of Context, 3 ACs:

> `DataSourceSection` currently renders two hardcoded dropdowns with a no-op onChange. …
> **Existing endpoints (reuse, zero new work):** `GET /api/v1/teams` — `GetTeams`
> (`api/team.go:1027`, registered `api_routing.go:250`) … **FE implementation (new hooks in
> `widgetStudio/workflow/api/`):** 1. `useGetProjectTeams` — `useQuery` calling … Query
> key: `workflowWidgetKeys.projectTeams()` … [~1,600 more chars]

**After** — one bullet:

> * APIs: `GET /api/v1/teams` (team list), `GET /api/v1/team/{teamId}/solutions` (apps —
>   `module_id` = `source.appId`). Full API contract: [Workflow Source API Contract](link)

**The acceptance criteria were identical.** Only the why shrank. Deep rationale — options
considered, alternatives rejected, design arguments — belongs in a linked doc; the ticket
keeps only the why that pins an AC.

## Step 5 — Grill the breakdown

Run a `/grilling` session on the proposed breakdown:

- **Delivery legibility** — Can a reader name what ships next without tracing dependencies?
  Is any deliverable named after a layer, module or phase rather than something a user sees?
- **Demoability** — Does every deliverable have the ticket that makes it actually work?
- **Framing** — Is there a Context bullet that constrains no AC? Is any Context longer than
  its AC list?
- **Granularity** — Tickets too coarse (unrelated changes bundled) or too fine (trivial
  standalone work)?
- **Dependencies** — Are they stated on each ticket, rather than left as a diagram to decode?
- **Missing work** — Walk the user journeys. Navigation between features, empty states,
  error states, permission boundaries.
- **Shared work** — If two deliverables need the same hook or component, which one owns it?

## Step 6 — Present and save

List deliverables in sequence order, tickets nested under each, unprefixed foundation
tickets last. Show every title and description.

Restate the parent story shape so the sequence lands where the team looks:

```markdown
## Acceptance Criteria
1. ...

## Deliverables Sequence
1. Datatable Tab
2. List View
...
```

Ask the user whether the breakdown should be saved to disk (e.g.
`documentation/docs/features/{feature-name}/`, alongside any existing ACs or decision logs).
Only save if they confirm.
