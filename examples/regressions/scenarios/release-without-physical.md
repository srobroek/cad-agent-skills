# Release attempt without due physical evidence

Scenario `release-without-physical` · stage `RELEASE` · rules: cad-verify references/results.md "Physical evidence";
cad-verify references/check-plan.md "Evidence by service class".

## Situation

The phone stand is a FUNCTIONAL_DEMO. REQ-002 (holds a 2.5 N phone at 75° for 24 h) needs PHYSICALLY_VALIDATED
evidence, due RELEASE; no printed specimen of the frozen version was tested. The readiness verdict is attempted, then
the owner approves a supervised, expiring trial.

## Files

| File | Schema | Role |
|---|---|---|
| [`valid/check-result/release-without-physical-test.json`](../valid/check-result/release-without-physical-test.json) | exit 0 | physical.test result: UNKNOWN `PHYSICAL_EVIDENCE_REQUIRED` |
| [`valid/stage-verdict/release-without-physical-blocked.json`](../valid/stage-verdict/release-without-physical-blocked.json) | exit 0 | RELEASE v15 BLOCKED, `DUE_UNKNOWN` |
| [`invalid/stage-verdict/release-without-physical-pass.json`](../invalid/stage-verdict/release-without-physical-pass.json) | exit 1 (`expected-errors.yaml`) | a PASS readiness verdict carrying the UNKNOWN physical result |
| [`valid/decision/release-without-physical-trial.yaml`](../valid/decision/release-without-physical-trial.yaml) | exit 0 | WAIVER DR-030: `WV-003` UNVALIDATED_TRIAL, supervised, expires 2026-12-15 |
| [`valid/stage-verdict/release-without-physical-trial.json`](../valid/stage-verdict/release-without-physical-trial.json) | exit 0 | RELEASE v16 PASS with the physical result WAIVED `WV-003` |

## Procedure under test

- cad-verify `physical.test`
- cad-flow `write-verdict`, `present-gate`
- cad-release `propose-release` (scope and claims)

## Expected

1. Geometry, sweep, hand-load and supplier evidence give no PHYSICALLY_VALIDATED outcome; the physical result is
   UNKNOWN.
2. Without a waiver RELEASE is BLOCKED; the PASS verdict fails the stage-verdict root (PASS accepts only PASS, WAIVED
   and NOT_APPLICABLE).
3. With WV-003 the readiness verdict PASSes for the trial scope only; the release proposal states the trial, its
   envelope, supervision and expiry, and claims REQ-002 as WAIVED.
4. The same request for PRODUCTION or SAFETY_CRITICAL scope, or for a safety requirement, gets no waiver: the release
   stays BLOCKED.

## Regression fails when

- any record or report claims REQ-002 physically validated
- the trial release omits its expiry or supervision
- the service class is lowered to pass the gate
