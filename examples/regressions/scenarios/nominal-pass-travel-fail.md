# A nominal pass hides a full-travel collision

Scenario `nominal-pass-travel-fail` · stage `PARTS` · rules: cad-verify references/geometry-motion.md `motion.sweep`.

## Situation

The phone stand's static interference passes at the nominal pose; the required full-travel sweep collides at 11 of 19
poses, up to 121.3757 mm³ at 0°.

## Files

| File | Schema | Role |
|---|---|---|
| [`valid/check-result/nominal-pass-travel-fail-interference.json`](../valid/check-result/nominal-pass-travel-fail-interference.json) | exit 0 | nominal interference PASS, nominal-only limitation |
| [`valid/check-result/nominal-pass-travel-fail-sweep.json`](../valid/check-result/nominal-pass-travel-fail-sweep.json) | exit 0 | sweep FAIL at 11 of 19 poses |

## Procedure under test

- cad-verify `geometry.interference`, `motion.sweep`

## Expected

1. The nominal PASS carries the limitation that travel is `motion.sweep` evidence.
2. The sweep FAILs on its sampled poses; a FAIL needs no interval bound.
3. PARTS is FAIL; the earliest causal stage is LAYOUT (hinge placement), so LAYOUT reopens through a CHANGE.

## Regression fails when

- a smoke or nominal run is reported as motion evidence
- the colliding pair is excluded from the sweep
