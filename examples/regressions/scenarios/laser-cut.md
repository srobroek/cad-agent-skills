# Laser tab-slot worst case and unsafe or unidentified stock

Scenario `laser-cut` · stage `PARTS` · rules: cad-verify references/process-checks.md `process.laser`. Covers
mfg-intent spec §10 items 10–11.

## Situation

Three laser-cut lamp parts: a panel with 3 mm tab-slot joints designed at +0.10 mm nominal clearance, a cover in PVC
sheet, and a shield whose coating is unidentified.

## Files

| File | Schema | Role |
|---|---|---|
| [`valid/check-result/laser-cut-process.json`](../valid/check-result/laser-cut-process.json) | exit 0 | laser result: tab-slot FAIL, PVC FAIL CRITICAL, coating UNKNOWN CRITICAL |

## Procedure under test

- cad-verify `process.laser`

## Expected

1. The tab-slot fit is evaluated at stock 3.0 ±0.2 mm and kerf 0.15 ±0.05 mm: −0.15 mm at the worst case, FAIL,
   despite +0.10 mm nominal.
2. PVC stock is FAIL with CRITICAL severity `material_unsafe`; no waiver applies.
3. The unidentified coating is UNKNOWN with CRITICAL severity `material_unidentified` until a retained source
   identifies it.
4. Even after the slots are widened, the fit needs a coupon: a `physical.test` obligation, PLANNED until cut.

## Regression fails when

- the nominal clearance passes the joint
- PVC is accepted with extraction
- the coating is assumed safe
