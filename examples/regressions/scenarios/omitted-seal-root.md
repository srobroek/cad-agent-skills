# An omitted seal root is UNKNOWN, and a seal list holds only reached paths

Scenario `omitted-seal-root` · stage `INTENT` · rules: cad-record references/validation.md Seal check (Outcomes, Naming,
Provenance) and Reporting; cad-flow references/worker-delivery.md Accept step 6.

## Situation

GROUND v1 passed and the owner approved it (DR-002). INTENT v2 failed, and its verdict records DR-002 as the approval of
its predecessor. The lead wrote the INTENT v3 `record.semantic` brief without DR-002. The brief therefore has no seal root,
and the checker's seal list is empty. The checker cannot read DR-002, because it is neither inventoried nor reached. It
also has no row category for the DR path. A row that names it in `files` would make Accept reject the delivery instead of
publishing the UNKNOWN result.

## Files

| File | Schema | Role |
|---|---|---|
| [`valid/check-result/omitted-seal-root-intent.json`](../valid/check-result/omitted-seal-root-intent.json) | exit 0 | INTENT v3 semantic UNKNOWN `MISSING_INPUT`, items 9 and 12: the brief lacks DR-002, so the seal list is empty and the rows cite the DR by its missing-root reference |

## Procedure under test

- cad-record references/validation.md Seal check, Outcomes: a seal root the brief lacks is UNKNOWN `MISSING_INPUT`, a brief
  defect, never PASS and never FAIL
- cad-record references/validation.md Seal check, Naming: a missing-root reference is the DR path that an inventoried
  verdict's `predecessor.approval` records, cited without a live read
- cad-flow references/worker-delivery.md Accept step 6: the lead recomputes the reached set from the brief's roots. Each
  seal-list line must lie in it with the same sha256. A `record.semantic` row may cite a seal-list path or a missing-root
  reference, and no other result may

## Expected

1. The v3 checker reports items 9 and 12 UNKNOWN `MISSING_INPUT`. Both findings name `hw/decisions/DR-002.yaml` and the
   v2 verdict that records it, and the remedy is a new brief that lists DR-002.
2. Accept publishes the result. `hw/decisions/DR-002.yaml` in item 12's `files` is a missing-root reference, because
   `predecessor.approval` of the inventoried `hw/versions/v2/stage-verdict.json` records it.
3. A seal list with a line that the lead's walk from the brief's roots does not reach, or with another sha256, is rejected
   at Accept, and the reason names that path. A holding item 12 whose list lacks a line of that set is rejected the same
   way. A seal-list path cited by a result of another family is rejected too.

## Regression fails when

- an omitted seal root is reported PASS or FAIL
- Accept rejects a `record.semantic` row that cites a missing-root reference, or accepts a DR path that no inventoried
  verdict records
- Accept accepts a seal-list line outside the reached set, a holding item 12 whose list omits a reached line, or a
  seal-list path in a result of another family
