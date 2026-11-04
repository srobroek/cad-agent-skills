# An unsupported process stays UNKNOWN; a legacy record is readable, not certified

Scenario `unsupported-process` · stage `PARTS` · rules: cad-verify references/process-checks.md `process.other`;
cad-record references/import.md. Covers mfg-intent spec §10 item 18.

## Situation

An imported legacy spring clip LMP-0009 keeps ASSEMBLY_ZERO under its import DR; its only operation is a hand-bent
wire OTHER process with no criteria.

## Files

| File | Schema | Role |
|---|---|---|
| [`valid/part/unsupported-process-legacy.yaml`](../valid/part/unsupported-process-legacy.yaml) | exit 0 | imported part: OTHER route, UNKNOWN intent fields |
| [`valid/check-result/unsupported-process-other.json`](../valid/check-result/unsupported-process-other.json) | exit 0 | process.other UNKNOWN `UNSUPPORTED_METHOD` |

## Procedure under test

- cad-verify `process.other`
- cad-record import (readable history, no certification)

## Expected

1. `process.other@LMP-0009.clip/bend` is UNKNOWN `UNSUPPORTED_METHOD`; no FDM, CNC or sheet criteria substitute.
2. The legacy record stays readable with its unknowns explicit; its legacy evidence counts for nothing at this stage.
3. PARTS is BLOCKED until the check plan gains a criterion list for the process (a SPECIFY change) or the clip is
   bought.

## Regression fails when

- another process's criteria are applied
- imported legacy results are cited as stage evidence
