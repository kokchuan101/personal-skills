# Coverage inventory — reusable subagent brief

Step 4 of the workflow. Delegate this to a read-only search agent (Explore or
general-purpose) before writing a single test. Paste the brief below, filling in
the target suites. Worked example: the FiltersTab suites
(`client/app/pages/app/reporting/widgetStudio/entities/workflow/SidePanel/tabs/FiltersTab/`
— `operators.test.ts`, `ComparatorField.test.tsx`, `CommaSeparatedInput.test.tsx`,
`FiltersTab.test.tsx`).

## The brief

> Read-only inventory task — do not edit any file, do not write tests.
>
> Target suites: `<paths or glob>`. Also read the modules under test and any
> shared fixture/mock files they import.
>
> Report, per test file:
>
> 1. **The full describe/it tree**, with every parameterized case
>    (`it.each`/table-driven/`@ParameterizedTest`) **expanded into its
>    individual case names**. Quote test names verbatim.
> 2. **Harness and setup idioms** — render/mount helpers, wrapper or provider
>    components, `beforeEach` stubs and global patches, mocking strategy (module
>    mocks vs stubs vs none), and how elements/values are queried or asserted.
> 3. **Shared fixtures** — what each fixture contains, field by field, and any
>    constraint on what shapes are realistic (which combinations the real
>    upstream can actually produce, which are synthetic conveniences).
> 4. **Exports with no direct test**, per module under test.
>
> Then list which exports/behaviors are already asserted and which are not.
> Do not propose new tests; report only what exists.

## Why each item

- **Expanded `it.each` rows** are the whole point: a collapsed `it.each` looks
  like one test and hides the twelve cases already covered. This expansion is
  what keeps new tests additive instead of accidentally restating existing
  assertions — duplicated assertions rot independently and double the cost of
  every future behavior change.
- **Harness idioms** make new tests land green on the first run instead of
  after three failures rediscovering the house setup (the stub some component
  needs, whether providers are used, whether globals are injected).
- **Fixture contents and their realism constraints** decide whether the new
  tests pin real behavior or a shape the server never sends — parity bugs live
  exactly in that gap, so a fixture invented for convenience can pass while
  production breaks.
- **Untested exports** are the honest gap list, and they usually map straight
  onto uncovered matrix cells.

## After the report lands

Map matrix cells → covered / uncovered, then decide per cell: extend an existing
`it.each` with a new row, add a new matrix-row group, or record it as not
unit-coverable (engine-side, another tab, covered by the live or E2E tier) and
say where it IS covered. That gap list ships with the final report.
