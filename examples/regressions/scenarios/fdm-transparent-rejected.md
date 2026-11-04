# A transparent part routed through FDM is rejected

Scenario `fdm-transparent-rejected` · stage `PARTS` · rules: cad-verify references/process-checks.md `optical.route`;
cad-select SKILL.md transparency rule.

## Situation

The phone stand's front lip window STD-0004 must keep the screen edge readable (REQ-006); its part record routes it
through FDM in a filament sold as "transparent".

## Files

| File | Schema | Role |
|---|---|---|
| [`valid/part/fdm-transparent-window.yaml`](../valid/part/fdm-transparent-window.yaml) | exit 0 | the part record: `transparency: TRANSPARENT`, FDM route; schema-valid |
| [`valid/check-result/fdm-transparent-optical.json`](../valid/check-result/fdm-transparent-optical.json) | exit 0 | optical.route FAIL, finding `fdm_optical` |
| [`valid/stage-verdict/fdm-transparent-parts.json`](../valid/stage-verdict/fdm-transparent-parts.json) | exit 0 | PARTS v7 FAIL: `DUE_FAIL`, review NOT_RUN |
| [`valid/part/fdm-transparent-remedy.yaml`](../valid/part/fdm-transparent-remedy.yaml) | exit 0 | the remedy part: laser-cut identified cast PMMA stock |

## Procedure under test

- cad-verify `optical.route`
- cad-flow `run-checks` step 4 (dependent families stop)
- cad-flow references/change-impact.md

## Expected

1. optical.route is FAIL whatever the filament name; its waiver policy is FORBIDDEN, so no waiver applies.
2. The PARTS verdict is FAIL; the review is UNKNOWN `NOT_RUN` because it reads the failed result; the gate is
   `NOT_PRESENTED`.
3. The remedy changes material and route, so INTENT reopens (the finding's recheck names INTENT) before a new PARTS
   attempt.
4. The remedy route still leaves optical transmission of the made window as a PLANNED `physical.test`.

## Regression fails when

- optical.route PASSes or is waived
- a capability or slicer setting is accepted as optical evidence
- the PARTS gate is presented
