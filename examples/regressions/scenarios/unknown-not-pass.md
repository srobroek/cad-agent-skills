# UNKNOWN never becomes PASS

Scenario `unknown-not-pass` · stage `LAYOUT` · rules: cad-verify references/results.md "Coverage"; cad-flow
references/stages.md `write-verdict`.

## Situation

The phone stand's hinge sweep ran its 19-pose grid; every sampled pose clears by at least 0.41 mm, but REQ-003 states
the 0.3 mm clearance over continuous travel and no computed bound covers the 18 intervals between poses.

## Files

| File | Schema | Role |
|---|---|---|
| [`valid/check-result/unknown-not-pass-sweep.json`](../valid/check-result/unknown-not-pass-sweep.json) | exit 0 | the sweep result: SAMPLED coverage, outcome UNKNOWN `INCOMPLETE` |
| [`valid/stage-verdict/unknown-not-pass-layout.json`](../valid/stage-verdict/unknown-not-pass-layout.json) | exit 0 | the correct LAYOUT v6 verdict: BLOCKED, `DUE_UNKNOWN` |
| [`valid/stage-verdict/unknown-not-pass-relabelled.json`](../valid/stage-verdict/unknown-not-pass-relabelled.json) | exit 0 | a wrong verdict that maps the UNKNOWN sweep to PASS; schema-valid, semantically FAIL |

## Procedure under test

- cad-flow `run-checks`, `accept-results`, `write-verdict`
- cad-verify `motion.sweep` (references/geometry-motion.md)
- cad-record semantic check 10, last item

## Expected

1. The sweep result is UNKNOWN with `unknown_code: INCOMPLETE` and lists every interval in `scope.unsampled`; it is
   never PASS.
2. The LAYOUT v6 verdict is BLOCKED with a `DUE_UNKNOWN` blocker on `motion.sweep@case.hinge-travel`; the gate is
   `NOT_PRESENTED`.
3. The relabelled verdict fails `record.semantic` (a results entry differs from its `result.json`), so it is never
   written; if found in `hw/`, every successor attempt BLOCKS.
4. Unblocking needs a new attempt with a per-interval bound or a finer grid; an approved waiver would map it to WAIVED
   in the verdict, never to PASS.

## Regression fails when

- the sweep result or the verdict entry says PASS
- the gate is presented for v6
- the unsampled intervals are dropped from the result or the report
