# Change impact

Owner: cad-flow. What a change, FAIL or MAJOR finding reopens, which evidence it makes stale, and when a result
may be carried forward. Stage `reopen` lists live in `assets/stages.yaml`.

## Triggers

- An edit to an approved or frozen input: use case, service class, risk, load, environment, requirement,
  criterion, interface, material, process, route, orientation, support, capability or calibration, source,
  geometry, part, assembly, case, method, sampling or check plan.
- A FAIL or MAJOR finding whose cause lies in an earlier stage, or a required review MINOR (`REVIEW_MINOR`) to close. A
  MINOR finding of a PASS verdict is a gate limit, never a trigger ([stages write-verdict](stages.md#write-verdict) step 6).
- A user request at a gate, or a worker proposal that touches approved scope.

## Earliest stage to reopen

| Changed element | Reopen |
|---|---|
| `product.yaml`: build mode, intake, service, risk, authority, release scope, rigor; `requirements.yaml` | GROUND |
| `hw/manufacturing/{materials,capabilities,selection}.yaml`, retained sources | INTENT |
| functional contracts in `interfaces.yaml`, `check-plan.yaml` and its cases | SPECIFY |
| `hw/concepts/**` | CONCEPT |
| the VARIANT gate choice | VARIANT |
| `assembly.yaml`, interface placement, PN allocation | LAYOUT |
| `hw/parts/<PN>/**` architecture, material, route, sourcing | PARTS |
| fits, tolerances, datums, fasteners, retention, support, setup, finish, inspection, drawing intent | DETAIL |
| BOM derivation method | BOM |
| an analysis method or solver deck, with unchanged cases | ANALYSIS |

The reopened stage R is the earliest row any changed element matches; it is the finding stage or one of its
`reopen` stages. An R later than the current stage reopens nothing: the change is input to R when it runs.

A lite build reopens only its own stages: `product.yaml`, `requirements.yaml` and `check-plan.yaml` with its cases →
GROUND; `assembly.yaml`, `interfaces.yaml`, PN allocation and `hw/parts/<PN>/**` → LAYOUT. A change that needs material,
sourcing, manufacturing, fit or strength scope is a [promotion to full](#promotion-to-full).

## Procedure

1. List the changed elements and the affected subjects (PN, component, IF, REQ, case, operation).
2. Take the earliest reopened stage R from the table.
3. Inside the open stage's own unapproved work (R is the current stage) → a new attempt with `supersedes`; no
   decision needed.
4. Otherwise write `hw/decisions/proposals/DR-###/change.yaml` (what changes, why, affected subjects, R, the
   evidence it makes stale) and present it. On explicit approval write the CHANGE decision with its `change`
   block (`reason`, `affected`, `reopen: R`, `baseline` the last accepted version, `superseded_failure` for a
   retry): `binding: PROPOSAL` before FREEZE is approved, `binding: FROZEN` (the frozen version, its manifest
   sha256 and verdict) after. A change without that decision is `UNAPPROVED_CHANGE`.
5. Run R as a new attempt whose verdict `supersedes` the attempt it replaces, then every later stage in order,
   each a new attempt with its gate presented again; none is skipped.
6. Older verdicts and decisions stay as history. A gate decision bound to a replaced verdict advances nothing;
   a waiver of a stale obligation lapses and needs a new approval.
7. Report the changed elements, R, every stale result as an absolute path, and the rerun order.

## Stale evidence

A result is stale when any input file it read changed or one of its subjects is affected. Stale results are
rerun, never relabelled. Rerun every affected structural, thermal and other analysis, tolerance stack, mechanism
sweep, mass and centre-of-mass result, support, setup, finishing and assembly plan, and every fit, insert,
strength or wear coupon or test. A physical test of a changed design no longer applies to it.

## Carry-forward

Before dispatch ([enumerate-obligations](stages.md#enumerate-obligations) step 6) the lead carries a check forward, instead of
rerunning it, when an accepted result of that check-plan check, published for an earlier version, holds all of:

- its `expected_obligations` equal this attempt's for the check, and every outcome is PASS or NOT_APPLICABLE;
- every input file it read is byte-identical to this attempt's inventory, and none of its subjects is affected;
- its family's method is the same at this stage and under both attempts' method profiles (`obligations.yaml`
  `method_profile`, absent in a full build): a family whose [cad-verify families](../../cad-verify/references/families.md)
  Method differs by stage (`interfaces.completeness`, `variant.integrity`) carries only between attempts of one stage;
- its family is none of the attempt audits `record.*` and `delivery.integrity`, which always rerun.

The `record.inputs` result of the new version records that byte comparison. The verdict entry is `CARRIED_FORWARD` with the result's
own binding and bytes, and `carried_from` names the original version, that `record.inputs` result and the reason. Without that explicit
assessment the check reruns; a FAIL or UNKNOWN result is never carried.

## Findings

- Cause inside the finding stage's own work → a new attempt of that stage.
- Cause in an earlier stage → this procedure with R from the table.
- MAJOR → the earliest causal owner gets a new attempt; a required review MINOR → close the finding, then recheck in a new
  attempt. A MINOR finding of a PASS verdict goes to the gate as a limit ([stages write-verdict](stages.md#write-verdict)
  step 6).
- Defective worker output → re-dispatch to the producing role ([delegation](delegation.md)), not a reopen.

## Promotion to full

A lite project becomes a full build only through this change; every lite byte stays history as written.

1. The owner asks for a full build. Write the CHANGE proposal ([Procedure](#procedure) step 4) with R GROUND: LITE → FULL,
   the lite baseline, the newly intended service, use, material, manufacturing and qualification scope, the affected
   subjects, the missing full fields and the new obligations. On explicit approval write the CHANGE decision with
   `baseline` the PASS LAYOUT version: `binding: FROZEN` (that version, its manifest sha256 and verdict) when a sealed lite
   package binds it, else `binding: PROPOSAL`.
2. Before any FULL GROUND freeze, keep every lite version, result, decision and release byte with its build mode and
   method profile, and check by sha256 that the baseline snapshot `hw/versions/<baseline>/record/` holds every lite part,
   model and assembly file. Then cut the live record over under that CHANGE: remove the lite part and model records from
   live `hw/parts/`, reset live `assembly.yaml`, `interfaces.yaml` and `check-plan.yaml` to the full template's empty
   shapes, and add the full template's missing collections without overwriting a file
   ([cad-record record-bootstrap](../../cad-record/references/record.md#record-bootstrap)). PN and component IDs stay
   reserved. Never freeze a FULL product beside a live LITE part, and never fill a material or other full field with a
   default.
3. Record the owner-sourced FULL build mode and the complete full intake
   ([cad-select intake-use-case](../../cad-select/references/selection.md#intake-use-case)), then run every full row from
   GROUND to RELEASE as new attempts with every check and gate; no full stage gets a PASS from lite evidence.
4. The lite STEP files, models and renders are concept input, cited by baseline snapshot path and sha256, and its sourced
   dimensions and decisions are traceability: never an approved selection, variant, manufacturing intent or evidence.
   CONCEPT and VARIANT may reuse that geometry as concept input; full LAYOUT publishes full part records. A lite gate
   decision approves no full verdict.
5. A lite result carries forward only by [Carry-forward](#carry-forward); every check that reads the product's build mode is
   stale.
6. Revision promotion and REV letters follow
   [cad-release promote-revisions](../../cad-release/references/release.md#promote-revisions) step 1: a lite package is never
   a revision baseline, and no consumed letter is reused.
7. The full release adds its own claims, inventory, approval, integrity and REV. Missing full, material or physical evidence
   blocks it; FreeCAD success supplies none of it.
