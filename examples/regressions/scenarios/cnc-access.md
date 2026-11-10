# CNC cutter, holder and workholding

Scenario `cnc-access` · stage `PARTS` · rules: cad-verify references/process-checks.md `process.cnc`. Covers
mfg-intent spec §10 item 9.

## Situation

The lamp arm LMP-0002 is milled from 6082 bar with sharp slot corners and a 28 mm pocket; the capability's smallest
cutter is 6 mm and the setup records no workholding.

## Files

| File | Schema | Role |
|---|---|---|
| [`valid/check-result/cnc-access-process.json`](../valid/check-result/cnc-access-process.json) | exit 0 | cnc result: radius and reach FAIL, workholding UNKNOWN |

## Procedure under test

- cad-verify `process.cnc`

## Expected

1. `#slot_corner` FAIL `internal_radius`: R1.0 against the R3.0 the smallest cutter leaves.
2. `#deep_pocket` FAIL `reach`: the holder collides 6 mm above the floor.
3. `LMP-0002.arm/mill` UNKNOWN `workholding_unknown`: a missing clamp plan is UNKNOWN, not PASS.
4. Limitations state that access came from geometry and the tool list, not a CAM toolpath.

## Regression fails when

- access is passed from collision-free CAD alone
- the missing clamp plan defaults to a vice
