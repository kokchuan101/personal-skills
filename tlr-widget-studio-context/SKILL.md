---
name: tlr-widget-studio-context
description: Load TLR widget studio context — glossary, spec, and task-relevant reference docs. Use when starting a session that involves reporting widgets, the widget studio, the workflow/IMM/TLR engines, or reporting-engine paths in the validator or Go server.
---

Load context for a TLR widget studio task. Reads the glossary and high-level spec (always), then selectively loads reference docs based on what the task involves.

## Guard

Check that the TLR docs exist in this repo:

```
documentation/docs/features/top-level-reporting/glossary.md
```

If the file does not exist, tell the user: "The TLR context docs are not present in this repo — this skill only works in the cbdev repository." Then stop.

## Step 1 — Load baseline context

Read both of these (always, regardless of the task):

1. `documentation/docs/features/top-level-reporting/glossary.md` — ubiquitous language, engine vocabulary, phases, doc map
2. `documentation/docs/features/top-level-reporting/widget-studio-spec.md` — architecture, definition schema, adapter mapping, cross-team contracts

Use the glossary vocabulary in all communication and code. Respect the _Avoid_ notes (e.g. say "workflow engine" not "legacy engine"; say "definition" not "schema" when referring to the stored shape).

## Step 2 — Load additional reference docs if the task needs them

Based on the user's task description, decide whether to also read any of these. Load silently — do not ask the user which docs to load.

| If the task involves... | Also read |
|---|---|
| Go server code in `server/src/checkbox/stats/`, the internal execution or field-discovery routes, or debugging what the workflow engine returns | `workflow-engine.md` |
| Database tables (`widgets`, `dashboards`, `dashboard_widgets`, `reporting_widgets`, `reporting_dashboards`, `reporting_dashboards_widgets`), migrations, or definition JSONB shapes | `reporting-data-model.md` |
| Filters, operators, the camelCase → engine operator map, or the filter builder UI | `filters-and-operators.md` |

All files are under `documentation/docs/features/top-level-reporting/`.

If the task doesn't match any row, skip this step — the baseline is enough.

## Step 3 — Confirm and proceed

Print one line listing which docs you loaded, e.g.:

> Loaded TLR glossary + spec + workflow-engine.md.

Then proceed with the task. Do not summarize the docs back to the user.
