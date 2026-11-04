# A waiver outside its scope

Scenario `waiver-out-of-scope` · stage `DETAIL` · rules: cad-record references/validation.md semantic check 10;
cad-flow references/gates.md "Waivers".

## Situation

In the next DETAIL attempt the fdm check fails two obligations: the waived front-face seam and a support contacting
the hinge bore. The lead maps the whole check to WAIVED under `WV-001`, and then proposes an unbounded waiver for the
bore.

## Files

| File | Schema | Role |
|---|---|---|
| [`valid/check-result/waiver-out-of-scope-fdm.json`](../valid/check-result/waiver-out-of-scope-fdm.json) | exit 0 | fdm result: front_face FAIL (covered) and hinge_bore FAIL (not covered) |
| [`valid/stage-verdict/waiver-out-of-scope-detail.json`](../valid/stage-verdict/waiver-out-of-scope-detail.json) | exit 0 | a wrong PASS verdict citing WV-001 for the whole check; schema-valid, semantically FAIL |
| [`invalid/decision/waiver-out-of-scope-unbounded.yaml`](../invalid/decision/waiver-out-of-scope-unbounded.yaml) | exit 1 (`expected-errors.yaml`) | a waiver for the bore with neither `expires` nor `until_change` |

## Procedure under test

- cad-flow `write-verdict` step 3
- cad-flow references/gates.md "Waivers" steps 1–2
- cad-record semantic check 10

## Expected

1. `write-verdict` rejects WV-001 for `process.fdm@STD-0001.body/print#hinge_bore` while composing the verdict. It covers no bore obligation; the entry remains FAIL and the verdict is FAIL with `DUE_FAIL`.
2. If a published verdict still maps the out-of-scope waiver, the successor's `record.semantic` item 10 fails it.
3. The bore waiver fails the decision root (a waiver needs `expires` or `until_change`); it is never written.
4. Even bounded, a bore waiver is refused: REQ-003 is mandatory and the support finding is MAJOR; the remedy is a
   DETAIL rework (reaming or reorientation).
5. A bare "ok, waive the bore too" is no approval of an exact waiver packet (cad-flow references/gates.md).

## Regression fails when

- a PASS verdict is written for v10
- WV-001 is widened in place instead of a new DR
- the bore obligation reaches the gate as WAIVED
