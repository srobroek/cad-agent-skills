# A MINOR required review fails the stage

Scenario `review-minor-fails-gate` · stage `CRITIQUE` · rules: cad-review references/rubric.md; cad-flow
references/stages.md `write-verdict`.

## Situation

The CRITIQUE review finds every due criterion met, with one MINOR finding: the arm lip edge radius is 0.3 mm against
the 0.5 mm the owner's preference asks.

## Files

| File | Schema | Role |
|---|---|---|
| [`valid/check-result/review-minor-critique.json`](../valid/check-result/review-minor-critique.json) | exit 0 | review result: verdict MINOR, result FAIL |
| [`valid/stage-verdict/review-minor-critique.json`](../valid/stage-verdict/review-minor-critique.json) | exit 0 | CRITIQUE v13 FAIL with review MINOR |
| [`invalid/stage-verdict/review-minor-pass.json`](../invalid/stage-verdict/review-minor-pass.json) | exit 1 (`expected-errors.yaml`) | a PASS verdict over the MINOR review |

## Procedure under test

- cad-review `review.mechanical_dfm`, references/rubric.md
- cad-flow `write-verdict` step 3, references/change-impact.md "Findings"

## Expected

1. The rubric grades MINOR (bounded, non-safety, non-mandatory); the review outcome is FAIL against the
   `review_verdict EQUALS PASS` criterion.
2. The CRITIQUE verdict is FAIL; a PASS verdict with a required MINOR review fails the stage-verdict root.
3. The finding is closed in a new DETAIL attempt and CRITIQUE reruns; nothing advances to FREEZE on a MINOR.

## Regression fails when

- the stage PASSes with an open MINOR
- the reviewer downgrades the finding to INFO to pass
