---
name: cad-review
description: Grades cad-agent-skills deliveries PASS, MINOR or MAJOR and runs the read-only mechanical and DFM review. Use for "review v12 for DFM", "grade this delivery".
---

# CAD Review

TRIGGER
+ a `cad-mech-dfm-reviewer` brief: "review v12 for mechanical and DFM", "critique the parts before freeze"
+ any worker grading its delivery `PASS|MINOR|MAJOR` → references/rubric.md only
- "review my design" in a lead session → cad-flow, which dispatches a reviewer unit; the lead never reviews itself
- measuring clearance, running a stack or a process check → cad-verify
- choosing a material, process or supplier → cad-select
- LITE design review → cad-verify [lite intent inspection](../cad-verify/references/families.md#lite-intent-inspection); no DFM review

## Workflow

1. Grading only (every role): LOAD the rubric the brief names in `resources.rubric`, references/rubric.md
   (`skill://cad-review/references/rubric.md`) → one verdict by the worst finding and the worst due outcome.
2. Reviewer: read the brief at its absolute path and the role reference it names; resources come only from the brief.
3. LOAD references/review.md (`skill://cad-review/references/review.md`) and run `review.mechanical_dfm` → separate
   mechanical and DFM coverage entries for every expected obligation of the version.
4. Write findings and grade with references/rubric.md → `PASS`, `MINOR` or `MAJOR`.
5. Compose the result document per review.md "Result document" → `implementation.kind: REVIEW`, every inspected file
   with its brief-inventory sha256, no commands.
6. Reply in the role reference's format with the document as one fenced `json` block → the lead transcribes it.

## Rules

MUST Stay read-only: run no program (on Codex, shell read commands only:
  [cad-flow agent roles — Roles](../cad-flow/references/agent-roles.md#roles)), and hash, validate, write, delegate or
  ask nothing; read only the brief's inventory copies and read-only paths.
MUST Keep mechanical and DFM coverage separate, and assess each part and operation by its own route kind.
MUST Cite every measured value with units and the inspected path it came from; never measure, estimate or invent one.
NOT Grade PASS from evidence the references/rubric.md Rules exclude.
NOT Weaken, reinterpret or replace an approved criterion, or redesign the subject: remedies are proposals.
NOT Treat a render, geometry or simulation as physical validation.
