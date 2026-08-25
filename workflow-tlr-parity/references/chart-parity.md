# Chart parity — matrix dimensions and TLR seams

Read after the user has named the chart type (pie, bar, line, …). Scope the
matrix to that type only; do not batch chart types into one matrix — their
legacy config surfaces and divergence decisions differ.

## Sources

- Legacy: `client/app/components/WidgetDefinitionEditor/` chart config +
  `constants.ts`.
- TLR: `entities/workflow/types.ts` (`WorkflowChartVisualization`).
- Engine arbiter: `server/.../stats/repository/` and
  `validator/src/reporting-engine/widgets/`.

## Matrix dimensions to recover

- **groupBy**: ≤2 levels (engine max); `period` bucketing for date fields;
  groupability limits — no approval pseudo-fields (engine limitation); only ONE
  list-typed variable per groupBy (unnest cross-product).
- **measure**: formula set (count/sum/avg/…); `field: null` = count(*);
  LP_* input types excluded from measure (multi-valued per assessment).
- **sort**: axis (label vs value) × order; **resultDisplayLimit**;
  **barPosition** (stacked / side-by-side — bar charts only).
- **labels/format**: option label overrides (e.g. AI Agent), number `format`
  (currency), date bucketing labels.

## TLR seams to test against

- `entities/workflow/normalizeWidgetDef.ts` — capability gating.
- `entities/workflow/schema.ts` visualization schema — validator mirror: pin
  via client tests, change via validator.
- SidePanel chart tabs.
- Fixture: `MOCK_CHART_VISUALIZATION` in `entities/workflow/mock/`.

## If the chart type is not built in TLR yet

Follow the decision fork's second branch: matrix + divergences first, then
`eng-workflow:tdd` red-green against the matrix rows, then return to steps
3–5 (live payload check, coverage inventory, grouped pinning) once green.
