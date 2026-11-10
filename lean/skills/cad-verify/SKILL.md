---
name: cad-verify
description: Plans and runs cad-agent-skills verification checks. Use for "check clearance over full travel", "is this printable", "tolerance stack", "plan the checks".
---

# CAD Verify

TRIGGER
+ "check clearance over the full travel", "does the hinge collide", "check the fit", "run the tolerance stack"
+ "is this part printable", "can this be machined", "mass and centre of mass", "will it hold 20 N"
+ cad-flow stage work `specify-requirements`, `plan-checks`, `confirm-cases` or `index-verification`
+ a cad-checker brief, or a check to rerun after a change
- choosing a material, process or supplier → cad-select
- independent mechanical and DFM critique → cad-review
- building, exporting or reading back geometry → cad-model
- approving, waiving or advancing a stage → cad-flow

## Workflow

1. Lead stage work: LOAD references/check-plan.md (`skill://cad-verify/references/check-plan.md`) and run the
   heading named by the procedure ID → `hw/requirements.yaml` judged with no byte changed, a `hw/check-plan.yaml`
   conforming to its cad-record prose root, confirmed cases or `hw/verification/vN/` with its bundle manifest.
2. Before any run, enumerate scope with check-plan.md "Expected scope" → every obligation `<family>@<anchor>`
   assigned to exactly one check, or UNKNOWN with a `SCOPE_GAP`.
3. LOAD references/families.md (`skill://cad-verify/references/families.md`) → each obligation's anchor, producer
   and method owner; in a lite build, from its "Lite build" section.
4. LOAD only the method references your obligations name:
   - references/geometry-motion.md (`skill://cad-verify/references/geometry-motion.md`) for interference,
     clearance, motion and assembly operations;
   - references/process-checks.md (`skill://cad-verify/references/process-checks.md`) for capability, optical,
     purchased and `process.*`;
   - references/fits.md (`skill://cad-verify/references/fits.md`) for fits and stacks;
   - references/mass-loads.md (`skill://cad-verify/references/mass-loads.md`) for mass and hand loads;
   - references/fea.md (`skill://cad-verify/references/fea.md`) for FEA;
   - the owner skill's heading for a family owned outside cad-verify.
5. Write and run your own locked program per references/results.md (`skill://cad-verify/references/results.md`)
   → one `result.json` per check that validates against the cad-record `check-result` root.
6. Fill every outcome, code, facet and finding per results.md → result FAIL, UNKNOWN, NOT_APPLICABLE or PASS by
   its aggregation rule.
7. Workers deliver per their role reference; the lead accepts and publishes per cad-flow.

## Rules

MUST Enumerate every component × operation × critical feature, interface, assembly operation, case, requirement and
  bought part before running; an excluded pair, pose or feature is a scope gap, never a smaller PASS.
MUST Keep a missing method, input, criterion or test UNKNOWN with its code; UNKNOWN, PLANNED and WAIVED never become
  PASS.
MUST Claim PHYSICALLY_VALIDATED only from a physical test of this version; geometry, renders, simulation, hand
  calculations and supplier data never establish it.
MUST Measure against the check-plan criterion unchanged; a better criterion, case or sampling is a change for the
  lead, never an edit inside a check.
DEFAULT Worst-case limits, full-travel grids and every occurrence pair, unless the approved check plan justifies less.
NOT Use a smoke, fast or nominal-pose run, a preview or another unit's program as evidence.
NOT Ship or run a checker helper from this skill; every program is the product's, under `.cad-work/<unit>/`.

OUTPUT  (lead report per check; a worker replies in its role reference's format)
L1 `<check>/<attempt> <family>: PASS|FAIL|UNKNOWN|NOT_APPLICABLE — <obligations> — <coverage FULL|SAMPLED|PARTIAL>`
   Unknowns — obligation, code, what unblocks it — only if non-empty
   Limits — sampled intervals, nominal-only scope, facets not established
CAP 120w clean · 250w with findings
