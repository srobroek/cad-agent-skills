# A plan family without a check enumerates as `<family>@product` and blocks

Scenario `unanchored-family` · stage `SPECIFY` · rules: cad-verify references/check-plan.md `plan-checks` and
"Expected scope" step 6.

## Situation

The SPECIFY check plan covers clearance, sweep, FDM, review and the physical hold test, but no access, operations,
interference, play, capability, fit, mass or load checks, although the requirements, interfaces and selected routes
make those families apply.

## Files

| File | Schema | Role |
|---|---|---|
| [`valid/check-plan/unanchored-family-plan.yaml`](../valid/check-plan/unanchored-family-plan.yaml) | exit 0 | the incomplete check plan; schema-valid |
| [`valid/check-result/unanchored-family-completeness.json`](../valid/check-result/unanchored-family-completeness.json) | exit 0 | check_plan.completeness FAIL naming the nine missing families |

## Procedure under test

- cad-verify `plan-checks`, `check_plan.completeness`
- cad-flow `enumerate-obligations` steps 2 and 5

## Expected

1. `check_plan.completeness@product` FAILs; SPECIFY is FAIL and its gate is not presented.
2. Had it passed, LAYOUT would enumerate `motion.access@product` and the other missing families as UNKNOWN
   `UNSUPPORTED_METHOD` with a `SCOPE_GAP`, never PASS.
3. Product-wide families (record, intake, completeness, index) use `@product` legitimately under their fixed check ids.

## Regression fails when

- an unanchored family passes or is silently skipped
- the checker invents a criterion for it
