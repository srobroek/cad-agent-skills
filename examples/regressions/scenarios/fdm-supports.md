# Reachable supports pass; supports on precision surfaces fail

Scenario `fdm-supports` · stage `PARTS` · rules: cad-verify references/process-checks.md `process.fdm`. Covers
mfg-intent spec §10 items 6–7.

## Situation

The base prints with supports under its non-critical underside and against the hinge bore; the arm's pivot boss sits
on a soluble support with no finishing; a clip prints with no support plan at all.

## Files

| File | Schema | Role |
|---|---|---|
| [`valid/check-result/fdm-supports-process.json`](../valid/check-result/fdm-supports-process.json) | exit 0 | fdm result: underside PASS, hinge_bore and pivot_boss FAIL, clip UNKNOWN |

## Procedure under test

- cad-verify "Expected scope" (one obligation per component × operation × feature)
- cad-verify `process.fdm`

## Expected

1. `process.fdm@STD-0001.body/print#underside` PASS: reachable support on a non-critical surface.
2. `#hinge_bore` FAIL `support_on_critical` (MAJOR): no operation finishes the bore.
3. `STD-0002.body/print#pivot_boss` FAIL: a soluble support alone does not make a precision surface pass.
4. `STD-0005.clip/print` UNKNOWN `REQUIRED_UNKNOWN` `support_unplanned`: a missing support plan is never "no supports".
5. The result is FAIL; PARTS cannot PASS until the route adds finishing with allowance, access and inspection, or
   reorients.

## Regression fails when

- the bore passes because supports are "removable"
- the clip passes with an assumed NONE strategy
- one obligation covers the whole part
