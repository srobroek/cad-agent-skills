# Stages

Owner: cad-flow. `assets/stages.yaml` (prose root `stage-policy`,
[cad-record prose contract](../../cad-record/references/prose-contract.md#stage-policy)) is the stage policy: order, predecessor, gate,
owner skill, procedure IDs, check families, due stage, waiver ceiling and reopen targets. This file says how to execute it and repeats no
policy row. Paths, IDs and the hash graph follow cad-record ([layout](../../cad-record/layout.json),
[record contract](../../cad-record/references/record.md), [contract index](../../cad-record/references/contract-index.md)).

## Reading the policy

| Field                  | Rule                                                                                                                                |
| ---------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| `order`, `predecessor` | One linear chain GROUND → RELEASE. A stage starts only after its predecessor has a PASS verdict and a satisfied gate.               |
| `gate`                 | `HUMAN`: an APPROVED decision ([gates](gates.md)). `REVIEW`: the verdict's required review is PASS. `NONE`: the PASS verdict alone. |
| `owner`                | The stage's accountable skill. Each stage-work ID loads the skill of its owner reference ([Stage-work procedure IDs](#stage-work-procedure-ids)). |
| `procedure`            | Run every ID in listed order; each ID is defined below.                                                                             |
| `checks[]`             | Families this stage enumerates: `ALWAYS` applies; `CONDITIONAL` applies when `condition` holds on the frozen inputs.                |
| `due`                  | Equal to the stage: the obligation is `expected` and resolves now. A later stage: it is listed in `future` as PLANNED.              |
| `waiver_policy`        | Family ceiling: `FORBIDDEN` is never waivable; `APPROVAL_REQUIRED` follows [gates Waivers](gates.md#waivers).                       |
| `reopen`               | Earlier stages a FAIL, MAJOR finding or change found at this stage may reopen ([change-impact](change-impact.md)).                  |
| `check_order`          | The fixed cost order of `run-checks`; its stop rule and publication walk it ([run-checks](#run-checks) steps 3–4).                  |

`safety` and `required_by` are not used: safety comes from the requirement. `release.integrity` belongs to no stage verdict; it is the
post-approval package check inside `publish-release`.

## Lead scratch

The lead writes and stages scratch only at the `.cad-work` paths of [cad-record's layout](../../cad-record/layout.json), never at the
`.cad-work/` root and never in a system temp directory such as `/tmp`: `.cad-work/lead/scratch/` for stage work before `freeze-inputs`
allocates `vN` (candidate, input and assessment lists), `.cad-work/stage-vN/` from then on, `.cad-work/lead/tmp/` for temporary files,
and its check program `.cad-work/lead/checks.py` with `checks.py.lock` ([run-checks](#run-checks) step 1). Its uv cache, `TMPDIR` and
validator follow [preflight](preflight.md#probe-tools) and [uv commands](agent-roles.md#uv-commands). Nothing in scratch is evidence.

Every byte the lead copies into `hw/` comes from `.cad-work/`, and every record file among them is validated there first: YAML loads
with [cad-record's YAML loader](../../cad-record/references/validation.md#yaml-loader), then a file of a schema root validates against it
with the [schema command](../../cad-record/references/validation.md#schema-validation) → exit 0, and a file of a prose root conforms to
its [prose-contract](../../cad-record/references/prose-contract.md) table → every field, type, pattern and rule holds. A file that fails
is corrected in scratch and never published. A bad file an earlier attempt already published follows
[cad-record validation — Earlier snapshots](../../cad-record/references/validation.md#earlier-snapshots).

## Lifecycle procedure IDs (owner cad-flow)

### resume-state

1. Confirm `.cad-work/lead/active/claim.yaml` names this session ([preflight](preflight.md)); otherwise stop.
2. Derive the chain from `hw/versions/v*/stage-verdict.json` and `hw/decisions/DR-*.yaml`
   ([cad-record's Versions](../../cad-record/references/record.md#versions)). The furthest approved stage is the last one whose PASS verdict
   has its gate satisfied and whose chain back to GROUND is unbroken and not reopened by a later CHANGE decision.
3. For its successor: a PASS verdict with an unsatisfied HUMAN gate → `present-gate`; an attempt without a verdict and with an unchanged
   input inventory digest → resume at its first unfinished step; a FAIL or BLOCKED verdict, or an unfinished attempt with changed inputs → a
   new attempt once the blocker is resolved.
4. `test -f` every path in `.cad-work/lead/preflight.json` → a missing path stops with `MISSING_RESOURCE`.

Output: the stage, its attempt `vN` or `new`, and the next procedure ID.

### freeze-inputs

1. Diagnose first: load every authored YAML file the attempt will inventory whose bytes differ from its latest `hw/versions/v*/`
   snapshot, or that has none, with [cad-record's YAML loader](../../cad-record/references/validation.md#yaml-loader), and validate it
   against its layout root: a schema root with the [schema command](../../cad-record/references/validation.md#schema-validation), a prose
   root against its [prose-contract](../../cad-record/references/prose-contract.md) table, every ID pattern included; log into
   `.cad-work/lead/tmp/`. Fix each failure in the record before step 2: the snapshot publishes these bytes
   ([Lead scratch](#lead-scratch)). This run is a diagnostic, never a result ([run-checks](#run-checks) step 7).
2. Allocate `vN`: the next integer above every `hw/versions/v*`, `hw/checks/v*` and `.cad-work/stage-v*`; `mkdir .cad-work/stage-vN` must
   succeed. Never reuse a number.
3. Write `.cad-work/stage-vN/input-inventory.txt`: every record file, part model, check plan and case, retained source and accepted decision
   the attempt reads, with its digest ([cad-record's Inventory digest](../../cad-record/references/record.md#inventory-digest)).
4. From now until the verdict, a changed inventoried byte is `SOURCE_MUTATION`: stop.

### build-version

1. Recompute the input digest; a mismatch is `SOURCE_MUTATION`.
2. When the stage lists `geometry.build`, build and read back the integrated definition from the snapshot
   ([cad-model's model contract](../../cad-model/references/model-contract.md) and [exports](../../cad-model/references/exports.md)) →
   geometry `PRESENT`. Otherwise record geometry `NOT_APPLICABLE` with the reason: GROUND to VARIANT "no assembly definition yet", CRITIQUE
   "reviews the predecessor version", RELEASE "payload is the FREEZE version".
3. Stage the snapshot and geometry under `.cad-work/stage-vN/` and copy them following
   [cad-model's Version build contract](../../cad-model/references/exports.md#version-build). Write the manifest last as
   `.cad-work/stage-vN/manifest.json` (prose root [version-manifest](../../cad-record/references/prose-contract.md#version-manifest)) and
   check it against that table → every field and rule holds; only then `test ! -e` and copy it to `hw/versions/vN/manifest.json`. A
   manifest that fails is corrected in scratch and never published, the rule of run-checks step 3.
4. An interrupted copy with unchanged inputs completes with identical bytes; changed inputs leave `vN` without a manifest, never resumed and
   never reused.

### enumerate-obligations

1. Evaluate every `checks[]` entry on the frozen inputs and on this attempt's units: those its stage work dispatched and those the
   [run-checks](#run-checks) step 1 split sends out. Record each false `condition` with its reason.
2. Take anchors from [cad-verify's check plan](../../cad-verify/references/check-plan.md). A family the check plan does not anchor uses
   `<family>@product`.
3. `due` equal to the stage → `expected`; a later `due` → `future`.
4. Write `hw/versions/vN/obligations.yaml` once, `test ! -e` first, before any check runs: `expected` and `future` obligation IDs and
   `inapplicable` entries `{family, reason}`. The verdict's `expected` and `future` equal it, and its `obligations` hashes it.
5. An expected obligation with no check-plan method or no available tool is UNKNOWN `UNSUPPORTED_METHOD`; never substitute a method.
6. Before any dispatch, apply [change-impact Carry-forward](change-impact.md#carry-forward) to every expected obligation; units go out only
   for the obligations it does not carry.

### run-checks

1. Split the obligations still to run by family. The lead runs `record.schema`
   ([cad-record validation](../../cad-record/references/validation.md)), `record.ownership`, `record.inputs` and `delivery.integrity`
   (hash, schema and ownership audits) with `producer_role: lead`; a `cad-mech-dfm-reviewer` unit runs `review.mechanical_dfm`; one
   `cad-checker` unit per group below runs every other family, a family no row names being a group of its own. Every unit whose families
   read no result of another pending family goes out in one batch ([delegation](delegation.md)); the rest go in the next batch, once the
   results they read are published.

   | Group     | Families                                                                                             |
   | --------- | ---------------------------------------------------------------------------------------------------- |
   | semantic  | `record.semantic`                                                                                    |
   | intake    | `intake.use_case`, `requirements.completeness`, `interfaces.completeness`, `check_plan.completeness` |
   | selection | `selection.*`, `material.properties`, `purchase.identity`, `variant.integrity`                       |
   | geometry  | `geometry.*`                                                                                         |
   | motion    | `motion.*`, `assembly.operations`, `fit.*`                                                           |
   | process   | `capability.route`, `optical.route`, `process.*`                                                     |
   | loads     | `mass.properties`, `load.*`                                                                          |
   | documents | `drawing.*`, `preview.render`                                                                        |

   While the batch runs, the lead runs its families on the frozen `vN` into `.cad-work/stage-vN/checks/<check>/<attempt>/` with one PEP 723
   program, `.cad-work/lead/checks.py`, locked beside it ([uv commands](agent-roles.md#uv-commands)), written once and changed only to
   fix a defect. Each lead result is COMMAND; its `source_files` are `source/checks.py` and `source/checks.py.lock`, copied into the bundle
   at publication ([cad-verify implementation kind](../../cad-verify/references/results.md#implementation-kind)). `delivery.integrity`
   runs with that program after the attempt's last merge-deliveries, over every triple logged in the attempt, and is published last.
2. Every stage result binds `VERSION` (vN and manifest digest). A worker's UNIT result is never stage evidence; a checker may read scratch
   first, then emits its VERSION result on the identical inventoried input.
3. Publish nothing unvalidated ([Lead scratch](#lead-scratch)): the lead writes its own results, and every check bundle manifest by
   [cad-record record — Bundle manifest](../../cad-record/references/record.md#bundle-manifest), in
   `.cad-work/stage-vN/checks/<check>/<attempt>/` first, and a worker's arrive through [worker-delivery](worker-delivery.md) acceptance.
   A file that fails is never published (the lead corrects its own in scratch; a worker's delivery is rejected). When the batch has
   returned and every unit is ACCEPTED, REJECTED or STOPPED, apply step 4, then publish the surviving results in `check_order` to
   `hw/checks/vN/<check>/<attempt>/`, each before the batch of any family that reads it is dispatched.
4. Stop rule, applied at acceptance in `check_order`: a FAIL or UNKNOWN in `record.*`, `geometry.build` or `geometry.readback` stops every
   later family; any other FAIL or UNKNOWN stops the families that read its result, artifacts or geometry. A stopped family's speculative
   result, the lead's or a unit's, stays unpublished in scratch, and `checks.py` records each stopped obligation in a lead result of its
   check: UNKNOWN `NOT_RUN` naming the blocking obligation. `delivery.integrity` runs after every family and stops none.
5. A rerun with unchanged inputs (interruption, timeout) is the next attempt `aN` of the same check with `supersedes`; a changed input, case
   or method is a new `vN`.
6. A result is `CARRIED_FORWARD` only under [change-impact Carry-forward](change-impact.md#carry-forward).
7. Fast or smoke runs stay in scratch as diagnostics and never enter a result.
8. A family whose subject a later procedure ID produces runs in that step; a family that reads such a subject or its result runs after
   it, at the end of the last such step ([cad-verify index-verification](../../cad-verify/references/check-plan.md#index-verification)).

### accept-results

1. Every expected obligation has one accepted outcome: PASS, NOT_APPLICABLE with its reviewer, the FAIL or UNKNOWN it ended with, or WAIVED
   when an APPROVED waiver `WV-###` covers that FAIL or UNKNOWN ([gates Waivers](gates.md#waivers)).
2. Every recorded sha256 of a published result, bundle manifest and derived bundle equals the bytes on disk; each file was validated
   against its schema root, or conformed to its prose root, before publication ([run-checks](#run-checks) step 3).
3. Result scopes cover exactly `obligations.yaml`: nothing missing, nothing extra.

### write-verdict

1. Compose the verdict as `.cad-work/stage-vN/stage-verdict.json`
   ([cad-record stage-verdict schema](../../cad-record/schemas/stage-verdict.schema.json)): `stage` and `version` vN;
   `version_manifest`, the path and sha256 of `hw/versions/vN/manifest.json`, and `obligations`, those of `obligations.yaml` once written;
   `policy` `{path, plugin}` from [preflight](preflight.md#preflightjson); `predecessor` `{stage, version, verdict, approval}`, the
   approved predecessor verdict and its gate DR
   ([cad-record decision bindings](../../cad-record/schemas/decision.schema.json#/$defs/binding));
   for a retry, `supersedes` `{version, verdict, reason}` of the attempt it replaces; `expected`, `future`, `results`, `bundles`,
   `waivers`, `blockers`; `review` with `required` true when the stage lists `review.mechanical_dfm` due now, plus its `verdict` and
   result file; `complete` true only when every expected obligation has an accepted outcome from a finished run; `time` in UTC.
2. Each `results[]` entry carries the check's result, except a FAIL or UNKNOWN check whose every non-PASS outcome an approved
   [cad-record waiver](../../cad-record/schemas/common.schema.json#/$defs/waiver) covers: its entry is `WAIVED` with that `waiver: WV-###`,
   and `waivers` lists `{id, decision}` for each waiver used.
3. Before writing, check the composed verdict against [cad-record validation](../../cad-record/references/validation.md#semantic-check)
   items 7 (waiver expiry at its `time`) and 10. A waiver that fails either covers nothing: its entries keep the check's result and
   `waivers` drops it.
4. `result`: PASS when complete, without blockers, every entry PASS, WAIVED or NOT_APPLICABLE and a required review PASS
   ([cad-review rubric](../../cad-review/references/rubric.md)). FAIL when a due obligation is evidenced FAIL or the required review is
   MINOR or MAJOR; BLOCKED otherwise.
5. `test ! -e hw/versions/vN/stage-verdict.json`, validate the composed file against the linked cad-record stage-verdict schema, and copy
   it there once → never rewritten.

### present-gate

HUMAN gates only: [gates](gates.md).

### advance

1. PASS verdict and satisfied gate → start the successor's procedure in the same turn, except after a gate reply that ends the session
   ([gates — Record a gate decision](gates.md#record-a-gate-decision) step 3).
2. FAIL or BLOCKED → the stop procedure; the next attempt or reopen follows [change-impact](change-impact.md).
3. Batches never cross a stage whose gate is not satisfied.

### dispatch-units and merge-deliveries

`dispatch-units`: [delegation](delegation.md). `merge-deliveries`: [worker-delivery](worker-delivery.md). Zero units is valid only when the
stage work names none.

## Stage-work procedure IDs

Before running an ID, LOAD the SKILL.md of the skill its owner reference belongs to; that reference defines the step under a heading with
the procedure ID.

| ID                     | Owner reference                                                                                   | Output                                                       |
| ---------------------- | ------------------------------------------------------------------------------------------------- | ------------------------------------------------------------ |
| `record-bootstrap`     | [cad-record record-bootstrap](../../cad-record/references/record.md#record-bootstrap)             | `hw/` from the template or an approved import                |
| `intake-use-case`      | [cad-select intake-use-case](../../cad-select/references/selection.md#intake-use-case)            | intake, service, risk, authority, release scope, requirements, preferences, assumptions, questions |
| `select-candidates`    | [cad-select select-candidates](../../cad-select/references/selection.md#select-candidates)        | candidates and the facts to source: one `cad-sourcing-researcher` unit per material family or capability without retained sources, all in one batch |
| `assess-selection`     | [cad-select assess-selection](../../cad-select/references/selection.md#assess-selection)          | `selection.yaml` assessment and recommendation               |
| `specify-requirements` | [cad-verify specify-requirements](../../cad-verify/references/check-plan.md#specify-requirements) | requirements checked complete; a gap is a change             |
| `specify-interfaces`   | [cad-design specify-interfaces](../../cad-design/references/layout.md#specify-interfaces)         | functional interface contracts                               |
| `plan-checks`          | [cad-verify plan-checks](../../cad-verify/references/check-plan.md#plan-checks)                   | `check-plan.yaml` checks and cases                           |
| `draft-variants`       | [cad-design draft-variants](../../cad-design/references/concepts.md#draft-variants)               | 2–4 concept units and their merged variants                  |
| `compare-variants`     | [cad-design compare-variants](../../cad-design/references/concepts.md#compare-variants)           | comparison of the unchanged variants                         |
| `define-layout`        | [cad-design define-layout](../../cad-design/references/layout.md#define-layout)                   | assembly tree, placements, joints, PN allocation, part units |
| `design-parts`         | [cad-design design-parts](../../cad-design/references/parts.md#design-parts)                      | part and sourcing units per PN group                         |
| `detail-parts`         | [cad-design detail-parts](../../cad-design/references/parts.md#detail-parts)                      | detail units: fits, tolerances, retention, process intent    |
| `draft-drawings`       | [cad-release draft-drawings](../../cad-release/references/drawings.md#draft-drawings)             | drawing bundles for the version                              |
| `derive-bom`           | [cad-release derive-bom](../../cad-release/references/bom.md#derive-bom)                          | BOM bundle for the version                                   |
| `confirm-cases`        | [cad-verify confirm-cases](../../cad-verify/references/check-plan.md#confirm-cases)               | every due case approved with an available method             |
| `index-verification`   | [cad-verify index-verification](../../cad-verify/references/check-plan.md#index-verification)     | verification bundle, then its `verification.index` check     |
| `promote-revisions`    | [cad-release promote-revisions](../../cad-release/references/release.md#promote-revisions)        | promoted part revisions before final checks                  |
| `propose-release`      | [cad-release propose-release](../../cad-release/references/release.md#propose-release)            | `hw/versions/vN/release-proposal.json`, then its `release.inventory` check |
| `publish-release`      | [cad-release publish-release](../../cad-release/references/release.md#publish-release)            | staged copy, `release.integrity`, final manifest             |

## Stop procedure

Stop on any condition below; the code is the verdict blocker when an attempt has a manifest. An outcome condition (`DUE_UNKNOWN`,
`DUE_FAIL`, `REVIEW_MAJOR`, `REVIEW_MINOR`) holds only for an outcome a check returned in a frozen attempt: a predicted outcome stops
nothing, so freeze the attempt, run its checks and write its verdict. Likewise a stage-work unit that ends REJECTED or lost
([delegation Lost and stalled units](delegation.md#lost-and-stalled-units)) stops nothing before a manifest: the facts it was to settle
stay typed UNKNOWN with owner and question in the record, and the lead freezes the attempt, runs its checks and writes its verdict, which
carries `DUE_UNKNOWN` or `DELIVERY_FAILED` where they fall due.

| Condition                                                                                                                                                                                                                  | Blocker                                                |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------ |
| foreign lead claim, no authorized recovery, a missing skill, schema, role or tool                                                                                                                                          | before a manifest: no verdict; else `MISSING_RESOURCE` |
| an installed file differs from what its contract states                                                                                                                                                                    | `INSTALLED_MUTATION`                                   |
| a due obligation with an UNKNOWN outcome; an UNKNOWN fact stops work only through the obligation where it falls due ([cad-select — Scope of each unknown](../../cad-select/references/selection.md#scope-of-each-unknown)) | `DUE_UNKNOWN`                                          |
| a due FAIL                                                                                                                                                                                                                 | `DUE_FAIL`                                             |
| missing result or incomplete producer                                                                                                                                                                                      | `MISSING_RESULT`                                       |
| scope or coverage gap                                                                                                                                                                                                      | `SCOPE_GAP`                                            |
| schema-invalid file                                                                                                                                                                                                        | `SCHEMA_INVALID`                                       |
| hash or reference mismatch, package mismatch                                                                                                                                                                               | `HASH_MISMATCH`                                        |
| inventoried input changed during the attempt                                                                                                                                                                               | `SOURCE_MUTATION`                                      |
| change without its approval                                                                                                                                                                                                | `UNAPPROVED_CHANGE`                                    |
| required review MAJOR                                                                                                                                                                                                      | `REVIEW_MAJOR`                                         |
| required review MINOR                                                                                                                                                                                                      | `REVIEW_MINOR`                                         |
| expired calibration or waiver                                                                                                                                                                                              | `EXPIRED`                                              |
| timeout                                                                                                                                                                                                                    | `TIMEOUT`                                              |
| rejected or failed worker delivery                                                                                                                                                                                         | `DELIVERY_FAILED`                                      |

A missing human approval is no blocker: the PASS verdict waits `WAITING_FOR_USER`.

1. Dispatch nothing new; stop this session's running units that depend on the failed work.
2. Retain every failed result, log, delivery and scratch file; delete nothing.
3. An attempt with a manifest gets its FAIL or BLOCKED verdict (`write-verdict`).
4. Report with the SKILL.md OUTPUT shape: stage, vN, blocker code, obligation, absolute paths, and the decision or information that unblocks
   it.
5. Write nothing more until the user answers or the cause is resolved; then follow [change-impact](change-impact.md).
