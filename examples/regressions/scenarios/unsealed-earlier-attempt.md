# A defective unapproved attempt is only re-hashed

Scenario `unsealed-earlier-attempt` · stage `LAYOUT` (lite) · rules: cad-record references/validation.md Attempt scope,
Earlier snapshots and items 2 and 12; cad-flow references/stages.md `write-verdict` and `accept-results`.

## Situation

The lite LAYOUT v3 attempt ended BLOCKED. The lead's verdict composer appended each two-obligation check once per obligation,
so the published v3 verdict lists two checks twice and repeats their blockers. A `__pycache__` file appeared under the v3
snapshot, and the v3 manifest does not list it. LAYOUT reruns as v4. Every v4 `record.semantic` then failed item 2 on the
immutable v3 verdict (round-7 smoke, lite Claude full cell, F2).

## Files

| File | Schema | Role |
|---|---|---|
| [`valid/stage-verdict/unsealed-earlier-attempt-reported.json`](../valid/stage-verdict/unsealed-earlier-attempt-reported.json) | exit 0 | the v3 verdict with duplicate rows and a `HASH_MISMATCH` blocker naming the unlisted path |
| [`valid/stage-verdict/unsealed-earlier-attempt-unreported.json`](../valid/stage-verdict/unsealed-earlier-attempt-unreported.json) | exit 0 | the same verdict without that blocker; schema-valid, and the unlisted path fails `record.schema` of v4 |

## Procedure under test

- cad-flow references/stages.md `write-verdict`: one entry per check and one blocker per code and subject, applied with the
  clearance items to the composed verdict before it is published
- cad-flow references/stages.md `accept-results`: an unlisted file under `hw/versions/vN/` is a `HASH_MISMATCH` blocker that
  names its exact path
- cad-record references/validation.md Attempt scope: an earlier attempt's verdict is never judged by a later attempt's items
- cad-record references/validation.md Earlier snapshots: RECORDED and REPORTED rows

## Expected

1. Today the composer writes each check once; a duplicate fails write-verdict and is never published.
2. A v3 verdict published before that rule stays as written. No approval binds it, so it is unsealed: v4's `record.semantic`
   re-hashes it and its results, and item 2 judges only v4's files. v4 is not failed by v3's duplicates.
3. `record.schema` of v4 gives the unlisted `__pycache__` path a `REPORTED` row. The row is PASS with the reported verdict,
   and its message names the v3 blocker; it is FAIL with the unreported verdict, as a change to history that v3 never
   reported. Neither row is record evidence.

## Regression fails when

- a later attempt fails an item on a failed or superseded attempt's verdict, results or snapshot content
- an unlisted path under an earlier snapshot passes without a `HASH_MISMATCH` blocker of its own attempt that names it
- a REPORTED row counts as record evidence or becomes an UNCHANGED baseline row
