# An approved waiver yields stage PASS

Scenario `waived-stage-pass` · stage `DETAIL` · rules: cad-flow references/gates.md "Waivers", references/stages.md
`write-verdict`.

## Situation

The base prints with a visible seam on its front face, failing the appearance requirement REQ-005 (priority MINOR,
waiver policy APPROVAL_REQUIRED). The owner approved a NON_SAFETY_DEVIATION waiver `WV-001` scoped to that one
obligation.

## Files

| File | Schema | Role |
|---|---|---|
| [`valid/decision/waived-stage-pass-dr.yaml`](../valid/decision/waived-stage-pass-dr.yaml) | exit 0 | WAIVER DR-021 with `WV-001` for `process.fdm@STD-0001.body/print#front_face` |
| [`valid/check-result/waived-stage-pass-fdm.json`](../valid/check-result/waived-stage-pass-fdm.json) | exit 0 | the fdm result: front_face FAIL, every other obligation PASS; result FAIL |
| [`valid/stage-verdict/waived-stage-pass-detail.json`](../valid/stage-verdict/waived-stage-pass-detail.json) | exit 0 | DETAIL v9 PASS with the fdm entry WAIVED `WV-001` |

## Procedure under test

- cad-verify `process.fdm` (references/process-checks.md)
- cad-flow `write-verdict` step 2
- cad-record semantic check 10 (existence, scope, expiry)

## Expected

1. The fdm `result.json` keeps `result: FAIL`; the producer never writes WAIVED.
2. The verdict maps only that check to `WAIVED` with `waiver: WV-001` and lists `{id: WV-001, decision: DR-021}`;
   result PASS.
3. Semantic check 10 passes: WV-001 is APPROVED, its obligation covers the only non-PASS outcome, `retained_outcome`
   equals FAIL, `until_change` has not occurred.
4. The gate packet lists WV-001 under limits; the verification row of REQ-005 is WAIVED, not PASS.

## Regression fails when

- the fdm result is rewritten as PASS or WAIVED
- the gate packet omits the waiver
- the verdict PASSes without the `waivers` entry
