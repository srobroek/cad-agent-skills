# An integrity result that cites scratch sources is rejected

Scenario `integrity-scratch-path` · stage `GROUND` · rules: cad-record references/validation.md item 12; cad-verify
references/families.md `delivery.integrity`.

## Situation

The lead's GROUND `delivery.integrity` result audits two merged files of the `ground-v1-semantic` checker unit. Its
input list and item table cite each merged file by the unit's `.cad-work/ground-v1-semantic/output/…` source path
instead of the `hw/` target it was copied to (round-6 smoke, Kiro full A/B cell: the approved GROUND table made every
later `record.semantic` fail item 12).

## Files

| File | Schema | Role |
|---|---|---|
| [`invalid/check-result/integrity-scratch-path-ground.json`](../invalid/check-result/integrity-scratch-path-ground.json) | exit 1 (`expected-errors.yaml`) | the integrity result citing the unit's scratch sources |
| [`valid/check-result/integrity-scratch-path-hw-targets.json`](../valid/check-result/integrity-scratch-path-hw-targets.json) | exit 0 | the same audit naming each `hw/` target and its sha256 |

## Procedure under test

- cad-flow references/stages.md `run-checks` (the lead's `delivery.integrity`, published last)
- cad-verify references/families.md `delivery.integrity`: the lead reads each `units.yaml` merge triple in scratch and
  records only unit, `hw/` target and sha256
- cad-record references/validation.md item 12 and its Reporting item table

## Expected

1. The lead validates the result before publication; the scratch-path version exits 1 and is corrected in scratch,
   never published.
2. The published result, its bundle manifest and its item table name each unit and `hw/` target with its sha256; no row
   names a `.cad-work/` path.
3. `record.semantic` item 12 holds for this result in every later attempt.

## Regression fails when

- an integrity result, artifact or item-table row names a `.cad-work/` path
- the lead publishes the scratch-path result and later attempts fail item 12 on immutable history
