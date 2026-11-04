# Check plan

Owner: cad-verify. The SPECIFY and ANALYSIS stage-work procedures, the expected-scope enumeration that [cad-flow
`enumerate-obligations`](../../cad-flow/references/stages.md#enumerate-obligations) takes its anchors from, and the
evidence each service class needs. Fields and paths are cad-record's ([requirements schema](../../cad-record/schemas/requirements.schema.json),
[check-plan prose root](../../cad-record/references/prose-contract.md#check-plan), [layout](../../cad-record/layout.json)); the per-family method, anchor and fixed check id
are in [families.md](families.md).

## specify-requirements

Input: the approved GROUND and INTENT record (`hw/product.yaml` service class, risk, authority and release scope;
`hw/manufacturing/selection.yaml`) and `hw/requirements.yaml`.

1. Keep requirements, preferences, assumptions and questions in their own collections; a preference never becomes a
   requirement and an open question is never a criterion.
2. Give every requirement `subjects`, sourced `criteria` (property, comparator, value, units, state, conditions),
   `methods`, `facets`, `due`, `safety`, `waiver_policy`, `source` and `owner`. A value the user or a source has not
   given stays a typed UNKNOWN with owner and question; never a default or a typical value.
3. Methods: `ANALYSIS` for geometry, process, stack, mass and load evidence; `INSPECTION` of the model or drawing for
   CAD_VALID, of a made article for PHYSICALLY_VALIDATED; `DEMONSTRATION` and `TEST` only for PHYSICALLY_VALIDATED.
4. For each requirement, list the facets relevant to it, chosen from those allowed for its service class and kind in
   the table below. A requirement need not list every allowed facet. If it omits a facet mandatory for its service
   class, that omission is a scope gap: the affected outcome is UNKNOWN, never PASS.
5. `due`: the earliest stage whose verdict needs the evidence (CAD_VALID from LAYOUT, GEOMETRICALLY_MANUFACTURABLE and
   PROCESS_PLANNED from PARTS, SIMULATION_SUPPORTED at ANALYSIS, PHYSICALLY_VALIDATED at RELEASE).
6. `safety: true` for kind SAFETY or REGULATORY and for any requirement whose failure the product risk names as a
   harm to people; then `waiver_policy: FORBIDDEN`. Otherwise `APPROVAL_REQUIRED` only where a waiver class in
   [cad-flow gates — Waivers](../../cad-flow/references/gates.md#waivers) can apply, else `FORBIDDEN`.
7. Validate `hw/requirements.yaml` against the requirements root → exit 0.

`requirements.completeness@product` is PASS only when: at least one requirement exists; the service class is KNOWN
with its DR; every requirement has KNOWN criteria, a method, its applicable facets and `due` per step 4; every
safety requirement is FORBIDDEN; every requirement kind outside the class's scope is listed in
`product.yaml` `release_scope.exclusions`. A consequential UNKNOWN criterion or service class is UNKNOWN
`REQUIRED_UNKNOWN`; omission of a facet mandatory for that service class is also a `SCOPE_GAP`. A missing method or
exclusion is FAIL.

### Evidence by service class

| Service class | Requirement kinds in scope | Facets | PHYSICALLY_VALIDATED for | `UNVALIDATED_TRIAL` |
|---|---|---|---|---|
| `APPEARANCE_CONCEPT` | APPEARANCE, MANUFACTURING, SAFETY, REGULATORY | CAD_VALID, GEOMETRICALLY_MANUFACTURABLE, PROCESS_PLANNED | none; no fit or function claim | not applicable |
| `FIT_ASSEMBLY` | + INTERFACE, SERVICE | all five | fit, clearance, play and assembly criteria | non-safety only |
| `FUNCTIONAL_DEMO` | + FUNCTIONAL, PERFORMANCE, ENVIRONMENTAL inside the demo operating limits | all five | every in-scope physical criterion | non-safety only |
| `WORKING_PROTOTYPE` | every kind | all five | every physical criterion | non-safety only |
| `PRODUCTION`, `SAFETY_CRITICAL` | every kind | all five | every physical criterion, with AUTHORITY evidence | never |
| `UNKNOWN` | none: `requirements.completeness` UNKNOWN `REQUIRED_UNKNOWN` | — | — | — |

A SAFETY or REGULATORY requirement is in scope at every class and never waived. Rigor never removes a facet.

## plan-checks

Input: the requirements from `specify-requirements`, `hw/interfaces.yaml` from
[cad-design layout — specify-interfaces](../../cad-design/references/layout.md#specify-interfaces), the stage policy
`assets/stages.yaml` of cad-flow and the installed tools in `.cad-work/lead/preflight.json`.

1. For every family that families.md marks `plan` and whose stage-policy condition can hold for this product (its
   requirements, interfaces, selected routes and sourcing), write at least one check in `hw/check-plan.yaml`.
2. Fill each check:
   - `subjects`, `components`, `operations`: the scope rule. `PRODUCT` or `ASSEMBLY` covers every scope item of
     the family; a named PN, interface, requirement or case covers only that item.
   - `criterion`: the requirement criterion, or the reference criterion as a `POLICY` source naming the
     cad-verify reference and heading; never a value the reference does not give.
   - `method`: the families.md method in words, naming the reference heading.
   - `facet`, `due`: the facet the method establishes; the first stage whose policy lists the family (RELEASE for
     `physical.test`).
   - `applicability`: APPLICABLE; CONDITIONAL with the stage-policy condition; or NOT_APPLICABLE with `reason` and
     `reviewed_by`.
   - `safety`: true when any covered requirement is safety.
   - `waiver_policy`: FORBIDDEN when the family ceiling, a covered requirement or `safety` says so.
   - `producer_role`: `cad-checker`; `cad-mech-dfm-reviewer` for `review.mechanical_dfm`.
3. Write the cases the methods need:
   - MOTION per functional motion: joints, sampling and steps per [geometry-motion.md](geometry-motion.md), `pairs: ALL`
     unless an interface excludes a pair with its reason, `play_extremes: true` when any member states play.
   - STACK per critical dimension chain: signed contributors, state and source; WORST_CASE unless a
     [JUSTIFIED_RSS](fits.md#fitstack) block and DR exist.
   - LOAD per stated load, with `material.basis_required: ALLOWABLE` for a strength criterion
     ([mass-loads.md](mass-loads.md#loadhand)).
   - FEA only when preflight found an installed solver and the user approved it ([fea.md](fea.md#loadfea)).
   - MASS once for the product: densities per made component and every purchased mass.
4. Check `hw/check-plan.yaml` against the cad-record `check-plan` prose table → every field and rule holds.

`check_plan.completeness@product` is PASS only when: every family step 1 names has a check; every requirement facet
maps to at least one check; every check criterion is KNOWN or a cited POLICY criterion; every case the methods need
exists with a justification for any SAMPLED or CONSTRAINED_PATH sampling; no two checks cover the same family and
scope item. An UNKNOWN criterion is UNKNOWN `REQUIRED_UNKNOWN`; a missing check, case or justification is FAIL.

## Expected scope

[`enumerate-obligations`](../../cad-flow/references/stages.md#enumerate-obligations) step 2 runs this on the frozen inputs of `vN`, before any check runs.
1. Scope items from the snapshot:
   - every made part (MAKE, MADE_TO_DRAWING), every component, every route operation that forms that component
     (operation `components`, or all components when absent) and every target feature or critical surface it forms:
     `<PN>.<component>/<operation>#<feature>`, plus `<PN>.<component>/<operation>` for the whole operation;
   - every BUY part `<PN>` and purchased component operation `<PN>.<component>/<operation>`;
   - every interface `IF-###` and each of its critical features (`IF-001#hinge_bore`); `assembly`; every assembly operation
     `op.<id>`; every case `case.<id>`; every requirement `REQ-###`.
2. A critical surface no operation forms, or a component no operation names, is a scope item of its own with
   outcome UNKNOWN `INCOMPLETE` and a MAJOR finding; it is never dropped.
3. Group repetitions only when geometry (equal component sha256 in the version manifest), material profile, route
   operations, feature and multiplicity are identical: anchor the first member in sorted order and list every member in
   the result's `scope.components`. Before geometry exists nothing groups. Similar or symmetric is not identical.
4. For each scope item, every family whose families.md anchor matches and whose stage condition holds gives one
   obligation `<family>@<anchor>`.
5. Assign each obligation to the one check whose family and scope rule cover it. No covering check → the obligation
   is `<family>@<anchor>` UNKNOWN `UNSUPPORTED_METHOD` with a `SCOPE_GAP` blocker; two covering checks → `SCOPE_GAP`
   and SPECIFY reopens ([cad-flow change impact](../../cad-flow/references/change-impact.md)).
6. Product-wide families (families.md anchor `@product`) give one obligation `<family>@product` under their fixed
   check id. For a `plan` family, `<family>@product` exists only when the plan does not anchor it and resolves UNKNOWN
   `UNSUPPORTED_METHOD`, never PASS.
7. Sort obligations by family in `check_order`, then anchor; write them to `obligations.yaml` per cad-flow.

## confirm-cases

Runs at ANALYSIS before `freeze-inputs`; it edits no case.

1. List every case a check names whose family the ANALYSIS policy lists due ANALYSIS.
2. Confirm each case: its bytes equal the approved SPECIFY check plan; every joint, contributor feature, subject,
   material profile and density it names exists in the current record; each property its criterion needs is KNOWN with
   the basis it requires; its method is available (an FEA solver at the exact version in `preflight.json`).
3. Write `.cad-work/lead/sessions/<stamp>/confirm-cases.yaml`: each case `CONFIRMED`, or blocked with
   `MISSING_INPUT`, `REQUIRED_UNKNOWN` or `UNSUPPORTED_METHOD` and the fact that unblocks it.
4. A case that must change is a change ([cad-flow change impact](../../cad-flow/references/change-impact.md), reopen SPECIFY); a blocked case
   leaves its obligations UNKNOWN with that code. A safety case is never postponed to a later stage.

## index-verification

Runs at ANALYSIS and FREEZE after `run-checks` has published this attempt's results.

1. Read the snapshot `requirements.yaml`, `obligations.yaml`, every accepted result of `vN` (VERSION or
   CARRIED_FORWARD) and every APPROVED WAIVER DR.
2. One row per requirement × subject × facet × criterion. `obligations`: the expected obligations whose outcomes
   name that requirement, subject and facet, plus the future obligations of that requirement and facet in
   - WAIVED when every FAIL or UNKNOWN obligation is covered by an approved waiver under [cad-flow stages — write-verdict](../../cad-flow/references/stages.md#write-verdict)
     step 2 (there is at least one); the row cites the waiver DR;
   - FAIL when any is FAIL; UNKNOWN (with the result's `unknown_code`) when any is UNKNOWN;
   - PLANNED when any is future;
   - NOT_APPLICABLE when all are; else PASS.
   A row with no obligation is UNKNOWN `INCOMPLETE` and a `SCOPE_GAP`. A row never cites the `verification.index`
   result, which reads this index.
3. `results` lists each cited `result.json` with its sha256; nothing is copied from a prior index, a CSV or a summary.
4. Write `hw/verification/vN/verification.yaml` (prose root `verification`, rows sorted by requirement, subject, facet),
   then `verification.csv` with the same rows and columns requirement, subject, facet, criterion, method, due,
   outcome, unknown_code, waiver, results; then the VERIFICATION bundle manifest.
5. Dispatch the `verification.index` check (references/families.md): an independent checker re-derives the rows and
   compares them; its result is published before `accept-results`.
