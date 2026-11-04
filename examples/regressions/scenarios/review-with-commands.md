# A REVIEW result with commands is rejected

Scenario `review-with-commands` · stage `CRITIQUE` · rules: cad-review references/review.md; cad-flow
references/worker-delivery.md "Read-only reviewer".

## Situation

The reviewer's reply JSON claims `implementation.kind: REVIEW` but also lists a command it says it ran to measure the
hinge clearance.

## Files

| File | Schema | Role |
|---|---|---|
| [`invalid/check-result/review-with-commands-critique.json`](../invalid/check-result/review-with-commands-critique.json) | exit 1 (`expected-errors.yaml`) | the transcribed reply block: REVIEW with `commands` |
| [`valid/check-result/review-with-commands-redispatch.json`](../valid/check-result/review-with-commands-redispatch.json) | exit 0 | the re-dispatched unit's read-only result `review/a2`, superseding a1 |

## Procedure under test

- cad-flow references/worker-delivery.md "Read-only reviewer" and "Accept" step 6
- cad-review `review.mechanical_dfm`, references/rubric.md

## Expected

1. The lead saves the block byte-identically; validation exits 1 (a REVIEW implementation forbids commands); the
   delivery is REJECTED, `DELIVERY_FAILED`.
2. The lead never strips the command or edits the JSON; it re-dispatches a new reviewer unit.
3. Executing anything is a reviewer boundary violation: MAJOR under references/rubric.md, whatever the measured value.
4. Only `review/a2` (REVIEW, inspected files only, `supersedes: a1`) enters the CRITIQUE verdict.

## Regression fails when

- the a1 block is repaired and accepted
- the reviewer's own measurement appears as evidence
- CRITIQUE advances on a1
