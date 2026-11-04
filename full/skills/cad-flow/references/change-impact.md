# Change impact

Owner: cad-flow. What a change, FAIL or MAJOR finding reopens, which evidence it makes stale, and when a result
may be carried forward. Stage `reopen` lists live in `assets/stages.yaml`.

## Triggers

- An edit to an approved or frozen input: use case, service class, risk, load, environment, requirement,
  criterion, interface, material, process, route, orientation, support, capability or calibration, source,
  geometry, part, assembly, case, method, sampling or check plan.
- A FAIL or MAJOR finding whose cause lies in an earlier stage, or a MINOR finding to close.
- A user request at a gate, or a worker proposal that touches approved scope.

## Earliest stage to reopen

| Changed element | Reopen |
|---|---|
| `product.yaml`: intake, service, risk, authority, release scope, rigor | GROUND |
| `hw/manufacturing/{materials,capabilities,selection}.yaml`, retained sources | INTENT |
| `requirements.yaml`, functional contracts in `interfaces.yaml`, `check-plan.yaml` and its cases | SPECIFY |
| `hw/concepts/**` | CONCEPT |
| the VARIANT gate choice | VARIANT |
| `assembly.yaml`, interface placement, PN allocation | LAYOUT |
| `hw/parts/<PN>/**` architecture, material, route, sourcing | PARTS |
| fits, tolerances, datums, fasteners, retention, support, setup, finish, inspection, drawing intent | DETAIL |
| BOM derivation method | BOM |
| an analysis method or solver deck, with unchanged cases | ANALYSIS |

The reopened stage R is the earliest row any changed element matches; it is the finding stage or one of its
`reopen` stages. An R later than the current stage reopens nothing: the change is input to R when it runs.

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
- its family's method is the same at this stage: a family whose [cad-verify families](../../cad-verify/references/families.md) Method
  differs by stage (`interfaces.completeness`, `variant.integrity`) carries only between attempts of one stage;
- its family is none of the attempt audits `record.*` and `delivery.integrity`, which always rerun.

The `record.inputs` result of the new version records that byte comparison. The verdict entry is `CARRIED_FORWARD` with the result's
own binding and bytes, and `carried_from` names the original version, that `record.inputs` result and the reason. Without that explicit
assessment the check reruns; a FAIL or UNKNOWN result is never carried.

## Findings

- Cause inside the finding stage's own work → a new attempt of that stage.
- Cause in an earlier stage → this procedure with R from the table.
- MAJOR → the earliest causal owner gets a new attempt; MINOR → close the finding, then recheck in a new attempt.
- Defective worker output → re-dispatch to the producing role ([delegation](delegation.md)), not a reopen.
