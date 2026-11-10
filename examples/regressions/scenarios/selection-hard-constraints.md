# Selection eliminates on hard constraints before preferences

Scenario `selection-hard-constraints` · stage `INTENT` · rules: cad-select references/selection.md `assess-selection`;
cad-verify `selection.feasibility`. Covers mfg-intent spec §10 items 1–5.

## Situation

A desk lamp near a 60 °C lamp head. Four assessments: an unconfirmed use case, a printable route that fails the heat
requirement, a phone stand where a printed polymer qualifies, and the phone-stand reference record
([examples/phone-stand](../../phone-stand/hw/requirements.yaml)) at INTENT, where the visible-face finish and the geometry
are criteria only a later stage evidences.

## Files

| File | Schema | Role |
|---|---|---|
| [`valid/selection/selection-use-case-unconfirmed.yaml`](../valid/selection/selection-use-case-unconfirmed.yaml) | exit 0 | intake DRAFT: every candidate CONDITIONAL; recommendation NONE, no route |
| [`valid/selection/selection-printable-excluded.yaml`](../valid/selection/selection-printable-excluded.yaml) | exit 0 | PLA EXCLUDED on REQ-010, PA6-CF and sheet hybrid CONDITIONAL, machined 6082 FEASIBLE and recommended |
| [`valid/selection/selection-polymer-qualified.yaml`](../valid/selection/selection-polymer-qualified.yaml) | exit 0 | printed PETG FEASIBLE on sourced evidence: FINAL with DR-008 |
| [`invalid/selection/selection-score-revives-excluded.yaml`](../invalid/selection/selection-score-revives-excluded.yaml) | exit 1 (`expected-errors.yaml`) | the excluded PLA candidate scored and recommended |
| [`valid/selection/selection-planned-finish.yaml`](../valid/selection/selection-planned-finish.yaml) | exit 0 | phone stand: REQ-005 `support_on_visible_faces EQUALS false` PLANNED due PARTS and REQ-001 to REQ-003 PLANNED due LAYOUT on each printed candidate; PETG and PLA FEASIBLE, ASA EXCLUDED on REQ-004; recommendation CONDITIONAL with the PLANNED outcomes as limits |
| [`invalid/selection/selection-planned-without-due.yaml`](../invalid/selection/selection-planned-without-due.yaml) | exit 1 (`expected-errors.yaml`) | the REQ-005 PLANNED outcome names no `due` stage |
| [`invalid/selection/selection-planned-due-intent.yaml`](../invalid/selection/selection-planned-due-intent.yaml) | exit 1 (`expected-errors.yaml`) | the REQ-005 PLANNED outcome is due at INTENT |

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
6. A criterion only a later stage evidences is PLANNED with its due stage and leaves the candidate FEASIBLE; a
   PLANNED outcome without `due`, or due at GROUND or INTENT, fails the selection root. An UNKNOWN names its question
   id.

## Regression fails when

- a score, price or availability revives an EXCLUDED or CONDITIONAL candidate
- a recommendation is FINAL before the intake is confirmed
- metal is chosen by default without the hard outcome that excludes the polymer
- a later-stage criterion (geometry, a process-plan property such as support contact) is assessed PASS or kept
  UNKNOWN at INTENT instead of PLANNED with its due stage
