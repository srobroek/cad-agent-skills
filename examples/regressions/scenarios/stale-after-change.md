# Evidence is stale after an approved change

Scenario `stale-after-change` · stage `PARTS` · rules: cad-flow references/change-impact.md "Stale evidence" and
"Carry-forward". Covers mfg-intent spec §10 item 13.

## Situation

After PARTS v7 the owner approved CHANGE DR-018: the arm STD-0002 moves from PETG to ASA. Geometry, assembly and cases
are unchanged. PARTS reruns as v9.

## Files

| File | Schema | Role |
|---|---|---|
| [`valid/decision/stale-after-change-change.yaml`](../valid/decision/stale-after-change-change.yaml) | exit 0 | CHANGE DR-018, `reopen: PARTS`, baseline v7 |
| [`valid/stage-verdict/stale-after-change-parts.json`](../valid/stage-verdict/stale-after-change-parts.json) | exit 0 | PARTS v9 PASS: geometry and motion results CARRIED_FORWARD with `carried_from`; material, capability, fdm and review rerun |
| [`valid/stage-verdict/stale-after-change-relabelled.json`](../valid/stage-verdict/stale-after-change-relabelled.json) | exit 0 | a wrong v9 verdict carrying `material.properties` forward; schema-valid, semantically FAIL |
| [`invalid/stage-verdict/stale-after-change-no-recheck.json`](../invalid/stage-verdict/stale-after-change-no-recheck.json) | exit 1 (`expected-errors.yaml`) | a v9 verdict with a CARRIED_FORWARD entry and no `carried_from` |

## Procedure under test

- cad-flow references/change-impact.md "Procedure"
- cad-verify `record.inputs` (references/families.md)
- cad-record semantic check 7

## Expected

1. Every result whose inputs include `STD-0002` `part.yaml` or its material profile is stale and reruns:
   material.properties, capability.route, process.fdm, review.
2. A geometry or motion result is CARRIED_FORWARD only with `carried_from` naming v7 and the v9 `record-inputs` result
   that shows its inputs byte-identical.
3. The relabelled verdict fails `record.inputs` (STD-0002 `part.yaml` differs), so its material entry is STALE and the
   verdict is BLOCKED, not PASS.
4. A CARRIED_FORWARD entry without `carried_from` fails the stage-verdict root.

## Regression fails when

- a stale result is relabelled or re-dated
- the v9 gate is presented on carried material evidence
- v7's approval is treated as approving v9
