# Why each rule exists

Keyed to SKILL.md sections. Read before skipping a step or "simplifying" a
rule — every entry below was paid for in a real parity session (workflow
filters).

## Matrix first (step 1)
Re-implementation is a spec-recovery problem: the legacy code IS the spec and
nobody wrote it down. Tests written without the matrix pin accidents, not
decisions.

## Engine as arbiter (step 1)
The UIs can both be wrong about the contract. Single-vs-multi user selection
came from the engine (`stats/repository/helper.go` exactly-one fields), not
from either UI — legacy made creator AND owners single; TLR follows the
engine instead.

## Divergence record before tests (step 2)
Without a numbered "do NOT fix" list, a future session or reviewer will
"correct" TLR back toward legacy and silently undo a decision. The
workflow-filters parity work shipped five deliberate divergences; each has
been questioned at least once since.

## Live verification (step 3)
Code reading lies. Two real bugs this step surfaced were invisible to unit
tests:
duplicate status names crashed the dropdown (an app version can legally hold
same-named statuses), and the validator's `customErrorMap` threw mid-parse,
aborting `z.union` on its first branch and 400ing every operator except
`contains` — unit tests called `safeParse` on the schemas directly and never
hit the glue. Assert payloads, not pixels: display labels legitimately differ
from stored values (shows "AI Agent", stores `"AI Assistant"`).

## Coverage inventory (step 4)
Duplicated assertions rot independently and double the cost of every behavior
change. Expanding `it.each` rows is the part that prevents accidental
duplication — the raw `it(` count undercounts by ~40%. Capturing harness
idioms is what makes new tests land green on the first run.

## Pinning idiom (step 5)
Grouping by matrix rows makes a failing test name the exact matrix cell that
broke. Full-contract assertions (list + order + control pairing) are what
distinguish the grouped tests from the existing single-function generic tests
— overlap stays structural, never literal, so nothing is a rewrite of a
passing test. Real fixture shapes matter because parity bugs live in the gap
between convenient synthetic data and what the server actually sends (SEL
arrives as `string` + options, not an "options" dataType).

## Red-green fork (decision fork)
Red-green is structurally impossible for behavior that already exists and
passes — pinning is post-hoc by definition. For unbuilt features the matrix
is what makes the red tests meaningful instead of guessed; without it TDD
pins your assumptions about legacy, not legacy.

## Plural studio route (middle tier)
The singular `/widget/.../studio` URL renders the LEGACY studio, which breaks
on TLR UUIDs — you'd be live-verifying the wrong implementation entirely.

## Tier trust (verification loop)
When a live check contradicts a unit test, suspect the environment first
(stale bundle, un-restarted service, wrong route) before suspecting either
test — every such contradiction seen so far has been environmental.
