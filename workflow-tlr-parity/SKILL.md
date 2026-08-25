---
name: workflow-tlr-parity
description: Verify and pin behavior parity when re-implementing a legacy Checkbox feature in the TLR widget studio (or any new vertical slice). Use whenever the task involves matching legacy behavior, "parity", characterization/pinning tests, migrating a legacy builder (filters, charts, columns) into TLR, planning pie/bar/line chart work, or setting up a verification loop for widget-studio changes — even if the user never says "parity". Produces a parity matrix, a divergence record, matrix-grouped tests, and a three-tier verification loop.
disable-model-invocation: true
---

# Legacy ↔ TLR Parity Verification

Workflow: matrix → divergence record → live check → coverage inventory →
matrix-pinned tests. Rationale for every rule: `references/rationale.md` —
read it before skipping or "fixing" any step.

## Decision fork (read first)

- **Behavior already exists in TLR** → *pinning* (characterization): tests
  come AFTER discovery. Follow all 5 steps.
- **Behavior not built yet** → do steps 1–2 first, feed the matrix rows to
  `eng-workflow:tdd` as the failing-test spec, return for steps 3–5 once
  green.

## The 5-step workflow

### 1. Build the behavior matrix from the legacy reference

- Sources: legacy builder `client/app/components/WidgetDefinitionEditor/`
  (e.g. `FilterSection/`, `constants.ts`); TLR studio
  `client/app/pages/app/reporting/widgetStudio/entities/workflow/`.
- One markdown table per behavioral dimension; one row per input kind, one
  column per implementation. Filter dimensions were: operator availability
  per dataType/fieldType; comparator control per operator; stored value shape.
- Add the engine as an arbiter column — its contract wins. Check
  `server/.../stats/repository/` and `validator/src/reporting-engine/widgets/`.
- Save in `documentation/docs/features/<feature>/no commit/` (untracked; do
  not commit unless the author asks). Structure:
  `assets/parity-matrix-template.md`.

### 2. Record deliberate divergences — before any test

Numbered entries: what differs, why, "do NOT fix", owner. Separately record
*parity-with-broken* items and pre-existing engine limitations, each with a
follow-up recommendation.

### 3. Verify the matrix live

Drive the real app (middle tier below) and assert
`POST /api/v2/reporting/widgets/:id/preview` payloads — at least one cell per
matrix row. Mark each row code-read vs live-verified.

### 4. Inventory existing test coverage

Delegate to a subagent using `references/coverage-inventory.md`: full
describe/it tree (`it.each` expanded), harness idioms, fixture contents,
untested exports. Map matrix cells → covered / uncovered. Extend, never
duplicate.

### 5. Pin the matrix with grouped tests

- Group describes by matrix rows (per system field, per field type), not by
  function. Worked example: `FiltersTab/operators.test.ts`. Idiom:
  ```ts
  const operatorControls = (candidate: DiscoveredField) =>
    operatorsForField(candidate).map((op) => [op, comparatorTypeFor(op, candidate)]);
  const systemField = (name: string) => {
    const match = MOCK_DISCOVERED_FIELDS.find((f) => f.field === name);
    if (!match) throw new Error(`MOCK_DISCOVERED_FIELDS has no ${name}`);
    return match;
  };
  const MEMBERSHIP_CONTROLS = [[INCLUDES, MULTI_SELECT], [EXCLUDES, MULTI_SELECT]];
  describe('system fields', () => {
    it.each(['status', 'statusCategory', 'creationMethod'])('offers %s membership …', (name) =>
      expect(operatorControls(systemField(name))).toEqual(MEMBERSHIP_CONTROLS));
  });
  ```
- Assert full contracts per named field: operator list (content AND order)
  plus the control each operator resolves to. Do not restate existing
  single-function assertions.
- Fixture rule — only shapes the server can return: SEL/RAD = dataType
  `string` + `options`; data-source select = `valueRef`, no options; LIST
  input types = option-less; approval pseudo-fields = `isSystemField`, no
  fieldType.
- Gate: scoped vitest green, `npm run tsc`, eslint on touched files.
- Report: files changed, test counts per group, and matrix cells NOT
  unit-coverable — with where each IS covered.

## Verification loop (three tiers)

### Inner — per edit
```bash
cd client && npx vitest run app/pages/app/reporting/widgetStudio/entities/workflow  # scope tighter as needed
docker compose exec -T validator npx vitest run --config ./vitest.config.cjs --coverage.enabled=false src/reporting-engine/widgets/...
```
Pre-push: `npm run tsc && npm run lint` in `client/`.

### Middle — interactive
- Use host dev server `http://localhost:8089`, NOT Kong (stale container
  bundle; symptom: no `widgetFields`/`preview` queries in React Query
  Devtools). Kong :8443 stays correct for runtime/public-assessment testing —
  see `testing-guidelines`.
- After validator edits: `docker compose restart validator`.
- Login `tester@checkbox.ai` / `a`; TLR flag is ON in local Unleash.
- Routes: list `/app/reporting/widget/list` (singular); studio
  `/app/reporting/widgets/<id>/studio` (plural).
- Test data: team 8 "Actual Form" / app 83 (all field types incl. LP_*,
  approval pseudo-fields); team 6 "Status" / app 30 (duplicate status names).
- Assert payloads, not pixels: `POST /api/v2/reporting/widgets/:id/preview`.

### Outer — CI
```bash
cd tools/playwright-tests && BASE_URL=http://localhost:8089 npx playwright test tests/e2e/reporting/ --project=chromium
```
Locally pass `BASE_URL=http://localhost:8089`; in CI use the default base
(flag comes from `unleash/backup-e2e-minimal.json`). One spec per feature
family: seed via API, assert preview payloads.

## Unit-test harness facts (client widget studio)

- vitest globals are on — never import `describe/it/expect/vi`.
- Design-system Dropdown needs
  `beforeEach(() => { Element.prototype.scrollIntoView = vi.fn(); })`.
- FiltersTab/ComparatorField-style components are controlled (`value`/
  `onChange`) — no FormProvider; use a `useState` harness wrapper when a
  value must round-trip.
- Shared fixture: `entities/workflow/mock/discoveredFields.ts`
  (`MOCK_DISCOVERED_FIELDS`). Extend only with server-real shapes; build
  one-off shapes with the local `field()` factory.
- Query by role/label; plain `render` — no providers, no `vi.mock`.
- `entities/workflow/types.ts` and `schema.ts` mirror validator files — never
  edit them from the client side of a parity task.

## Chart parity

Ask the user which chart type needs parity (pie, bar, line, …) before doing
anything — each type has its own legacy config surface and matrix rows. Then
read `references/chart-parity.md` for the matrix dimensions and TLR seams.

## Bundled resources

- `assets/parity-matrix-template.md` — copy-paste skeleton for step 1.
- `references/coverage-inventory.md` — subagent brief for step 4.
- `references/chart-parity.md` — chart matrix dimensions and TLR seams; read
  after the user names the chart type.
- `references/rationale.md` — the why behind each rule; read before deviating.
