# Selection eliminates on hard constraints before preferences

Scenario `selection-hard-constraints` · stage `INTENT` · rules: cad-select references/selection.md `assess-selection`;
cad-verify `selection.feasibility`. Covers mfg-intent spec §10 items 1–5.

## Situation

A desk lamp near a 60 °C lamp head. Three assessments: an unconfirmed use case, a printable route that fails the heat
requirement, and a phone stand where a printed polymer qualifies.

## Files

| File | Schema | Role |
|---|---|---|
| [`valid/selection/selection-use-case-unconfirmed.yaml`](../valid/selection/selection-use-case-unconfirmed.yaml) | exit 0 | intake DRAFT: every candidate CONDITIONAL; recommendation NONE, no route |
| [`valid/selection/selection-printable-excluded.yaml`](../valid/selection/selection-printable-excluded.yaml) | exit 0 | PLA EXCLUDED on REQ-010, PA6-CF and sheet hybrid CONDITIONAL, machined 6082 FEASIBLE and recommended |
| [`valid/selection/selection-polymer-qualified.yaml`](../valid/selection/selection-polymer-qualified.yaml) | exit 0 | printed PETG FEASIBLE on sourced evidence: FINAL with DR-008 |
| [`invalid/selection/selection-score-revives-excluded.yaml`](../invalid/selection/selection-score-revives-excluded.yaml) | exit 1 (`expected-errors.yaml`) | the excluded PLA candidate scored and recommended |

## Procedure under test

- cad-select `intake-use-case`, `select-candidates`, `assess-selection`
- cad-verify `selection.feasibility`, `intake.use_case`

## Expected

1. An unconfirmed use case leaves every candidate CONDITIONAL: the recommendation is NONE, names no route and lists the
   open question with its owner as a limit.
2. A printed route that fails a hard requirement is EXCLUDED; a missing capability or creep datum keeps a candidate
   CONDITIONAL.
3. A machined, sheet, purchased or hybrid alternative is assessed on the same hard outcomes; only FEASIBLE candidates
   are ranked.
4. A qualified polymer becomes the FINAL primary when its sourced evidence meets every hard criterion and the owner
   approves it.
5. A preference score on an EXCLUDED candidate fails the selection root; `selection.feasibility` FAILs any record that
   ranks it.

## Regression fails when

- a score, price or availability revives an EXCLUDED or CONDITIONAL candidate
- a recommendation is FINAL before the intake is confirmed
- metal is chosen by default without the hard outcome that excludes the polymer
