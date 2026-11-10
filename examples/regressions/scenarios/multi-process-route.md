# A multi-process route gets per-operation checks

Scenario `multi-process-route` · stage `DETAIL` · rules: cad-verify references/process-checks.md "Route kind to
family", `process.secondary`. Covers mfg-intent spec §10 items 12 and 17.

## Situation

The lamp pivot bracket LMP-0007 is milled, drilled, reamed and anodised, with a purchased iglidur bushing pressed into
the reamed bore.

## Files

| File | Schema | Role |
|---|---|---|
| [`valid/part/multi-process-bracket.yaml`](../valid/part/multi-process-bracket.yaml) | exit 0 | the part: CNC, DRILLING, REAMING, COATING and PURCHASED operations |
| [`valid/check-result/multi-process-secondary.json`](../valid/check-result/multi-process-secondary.json) | exit 0 | secondary result: drill PASS, ream allowance FAIL, coating compensation FAIL |

## Procedure under test

- cad-verify "Expected scope" and "Route kind to family"
- cad-verify `process.secondary`, `capability.route`, `purchase.identity`

## Expected

1. Obligations: `process.cnc` for the mill and its pivot_bore feature; `process.secondary` for drill, ream and
   anodise; `capability.route` for each made operation; `purchase.identity@LMP-0007.bushing/bushing`. No `process.fdm`
   obligation exists.
2. The ream FAILs: the mill finishes the bore at size, leaving no reaming stock.
3. The anodise FAILs: the 20–30 µm diametral build-up on the H7 bore is not compensated or masked.
4. The purchased bushing gets identity, rating and interface checks only, with its pv rating UNKNOWN until sourced.

## Regression fails when

- an FDM item is applied to the bracket
- one route-level obligation covers all operations
- the bushing gets made-part process checks
