# Missing calibration is UNKNOWN; a planned test stays PLANNED

Scenario `planned-physical-test` · stage `ANALYSIS` · rules: cad-verify references/results.md "Physical evidence";
references/check-plan.md `index-verification`. Covers mfg-intent spec §10 item 16.

## Situation

The phone stand passes ANALYSIS; REQ-002's physical hold test is due at RELEASE. Separately, at PARTS the arm's
hole-compensation calibration had expired.

## Files

| File | Schema | Role |
|---|---|---|
| [`valid/check-result/planned-physical-capability.json`](../valid/check-result/planned-physical-capability.json) | exit 0 | capability.route: arm UNKNOWN `REQUIRED_UNKNOWN` (expired calibration) |
| [`valid/stage-verdict/planned-physical-analysis.json`](../valid/stage-verdict/planned-physical-analysis.json) | exit 0 | ANALYSIS v11 PASS with `physical.test@REQ-002` in `future` |
| [`valid/verification/planned-physical-index.yaml`](../valid/verification/planned-physical-index.yaml) | exit 0 | the correct index: REQ-002 PHYSICALLY_VALIDATED row PLANNED |
| [`valid/verification/planned-physical-invented.yaml`](../valid/verification/planned-physical-invented.yaml) | exit 0 | a wrong index: that row PASS citing the hand-load result; schema-valid, semantically FAIL |

## Procedure under test

- cad-verify `capability.route`, `index-verification`, `verification.index`
- cad-flow `enumerate-obligations` step 3

## Expected

1. An expired calibration the criterion relies on gives UNKNOWN `REQUIRED_UNKNOWN`, never a pass on the old calibration.
2. At ANALYSIS the physical obligation is future: listed in `future`, absent from `expected` and from every result.
3. Its verification row is PLANNED with no results; the hand-load row is SIMULATION_SUPPORTED PASS.
4. The invented row fails the `verification.index` check: a SIMULATION_SUPPORTED result never fills a
   PHYSICALLY_VALIDATED row.

## Regression fails when

- a PLANNED obligation appears as PASS anywhere
- the expired calibration is carried
