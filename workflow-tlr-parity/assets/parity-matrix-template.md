# Parity matrix — fill-in template (legacy ↔ TLR)

Save as `documentation/docs/features/<feature>/no commit/parity-<area>-legacy-vs-tlr.md`.
Untracked working doc — do not commit unless the author asks.

Open with one line stating the feature/area, the branch/tree verified, and the date — a
matrix with no provenance gets trusted longer than it deserves.

**Status legend** (use in every Parity cell):

- ✅✅ live-verified — driven in the browser, preview payload asserted.
- ✅ code-read / unit — traced in source; branches may be dead or flag-gated.
- ⬜ unverified — inferred or remembered. Do not trust.

## 1. <Behaviour> availability per field kind

*One row per input kind (dataType / inputType), one column per implementation,
plus a Parity column recording how the row was confirmed. Cite exact legacy and
TLR source paths with line ranges above the table — the next session re-reads
them, and legacy is the spec. Add the engine/validator as a third arbiter column
whenever it disagrees with either UI: its contract wins (single-vs-multi user
fields came from `helper.go`, not from legacy UI).*

Legacy source: `client/app/components/WidgetDefinitionEditor/<Section>/<File>.tsx:NN-NN`
+ `WidgetDefinitionEditor/constants.ts:NN-NN`.
TLR source: `client/app/pages/app/reporting/widgetStudio/entities/workflow/.../<file>.ts`.
Engine/arbiter: `server/.../stats/repository/helper.go:NN`,
`validator/src/reporting-engine/widgets/<schema>.ts`.

| Field kind (dataType / inputType) | Legacy | TLR | Engine accepts | Parity |
|---|---|---|---|---|
| `<dataType>` (`<INPUT_TYPE>`, …) | <ordered list> | <TLR constant> — same N, same order | all N | ✅✅ (<what you saw>) |
| `<INPUT_TYPE>` (arrives as `<wire shape>`) | <legacy path> | <TLR predicate> | <narrower set> | ✅ unit (`<suite>.test.ts`) |
| `<kind>` | excluded (`<file>:NN`) | <same/different split> | — | ⚠ divergence #N |

## 2. Control per <behaviour>

*Second dimension: given the offered operator/behaviour, which comparator
control renders and what value shape gets stored. Record the literal stored and
transmitted value — `[0]` as a JSON number, `"YYYY-MM-DD - YYYY-MM-DD"` — not a
prose description: the stored shape is where a migration silently breaks saved
definitions and the engine 400s. Last column records the live check per cell.*

| Case | Legacy | TLR | Live check |
|---|---|---|---|
| `<operator>` | <legacy control>, stores `<literal>` | <TLR control>, stores `<literal>` | ✅ code + unit |
| `<operator>` | <legacy control> | <TLR control> (schema widened — engine accepts both) | ✅✅ payload `<literal json>` |

## 3. Deliberate divergences (do NOT "fix")

*Numbered so reviewers and future sessions can cite them. Each entry: what
differs, why, and who decided. Unnumbered/undocumented divergence is what gets
"corrected" back toward legacy six months later, undoing the decision.*

1. **<Name>** — legacy does X; TLR ships Y because <reason / decision owner>.
2. **<Name>** — legacy follows its own UI; TLR follows the **engine**
   (`server/.../stats/repository/helper.go`) because the engine's contract wins.
   Verified live: <observed behaviour>.
3. **<Name>** — out of scope; no known workflow app relies on it.

## 4. Known engine limitations (pre-existing, shared with legacy)

*Separate bucket: things broken on BOTH sides. Keeping them broken is
parity-with-broken, not a regression — say so explicitly, name the Go/validator
file and error, and end with a follow-up recommendation so it doesn't get
re-diagnosed every session.*

<Behaviour> on `<field kind>` <fails how> because <root cause, file:line> —
`<actual error text>`. Legacy offers the same field and fails identically, so
TLR keeping it is parity-with-broken.
**Recommendation:** <engine fix> or <exclude in both builders> as a follow-up.

## 5. Bugs found & fixed during live verification (<date>)

*Only bugs the live check surfaced that code reading and unit tests missed —
this section justifies step 3 existing. Record the invisible-to-unit-tests
reason for each, plus the regression test added.*

1. **<Symptom>** — <root cause>. Fixed in `<file>`, + unit test.
2. **<Symptom>** — <root cause>. Invisible to unit tests because they call
   `<inner function>` directly while the live path goes through `<glue>`.
   Fixed in `<file>`, + N regression tests. **This blocked <AC>.**

## 6. Verification loop

*Write the loop into the doc, not just into the session — the next person
inherits the traps instead of paying a session to rediscover them. Include the
host-dev-server vs Kong trap, restart commands, login, routes, and which seeded
team/app is rich enough to exercise every row.*

### Inner (seconds, per edit)
- Client: `cd client && npx vitest run <scoped path>`
- Validator: `docker compose exec -T validator npx vitest run --config ./vitest.config.cjs --coverage.enabled=false <path>`
- Pre-push: `npm run tsc && npm run lint` in `client/`.

### Middle (interactive, per feature)
- Host dev server `http://localhost:8089`, **not** Kong (stale container bundle).
- After validator edits: `docker compose restart validator`.
- Login `tester@checkbox.ai` / `a`; TLR flag ON in local Unleash.
- Studio route: `/app/reporting/widgets/<id>/studio` (plural).
- Test data: team <n> "<app>" / app <n> — covers <field kinds>.
- Assert `POST /api/v2/reporting/widgets/:id/preview` payloads, not pixels.

### Outer (CI regression)
`cd tools/playwright-tests && BASE_URL=http://localhost:8089 npx playwright test tests/e2e/reporting/ --project=chromium`
(spec: `<name>.spec.ts` — seeds via API, asserts preview payloads, ~Ns/run.)

## 7. Live-verified checklist results (manual verification)

*Mirror the feature's manual-verification steps one row each, and log what you
actually drove plus the literal payload observed — not "works". This table is
the evidence behind every ✅✅ above; an empty log means the matrix is still only
a code read. Row numbers let the PR description point at specific evidence.*

| # | Step | What was driven | Payload observed | Result |
|---|---|---|---|---|
| 1 | <manual verification step> | <interaction> | `<literal /preview body>` | ✅✅ (<row counts>) |
| 2 | <step> | <interaction> | `<literal>` | ✅✅ after <fix> (<before> → <after>) |
| 3 | <step> | <interaction> | `<literal>` | ✅ FE; ⚠ engine <limitation> (§4, pre-existing) |
