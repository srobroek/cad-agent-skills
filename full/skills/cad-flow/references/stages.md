# Stages

Owner: cad-flow. `assets/stages.yaml` (root `stage-policy`) is the stage policy: order, predecessor, gate, owner
skill, procedure IDs, check families, due stage, waiver ceiling and reopen targets. This file says how to execute
it and repeats no policy row. Paths, IDs and the hash graph follow cad-record ([layout](../../cad-record/layout.json),
[record contract](../../cad-record/references/record.md), [contract index](../../cad-record/references/contract-index.md)).

## Reading the policy

| Field | Rule |
|---|---|
| `order`, `predecessor` | One linear chain GROUND → RELEASE. A stage starts only after its predecessor has a PASS verdict and a satisfied gate. |
| `gate` | `HUMAN`: an APPROVED decision ([gates](gates.md)). `REVIEW`: the verdict's required review is PASS. `NONE`: the PASS verdict alone. |
| `owner` | The skill to LOAD for the stage-work procedure IDs. |
| `procedure` | Run every ID in listed order; each ID is defined below. |
| `checks[]` | Families this stage enumerates: `ALWAYS` applies; `CONDITIONAL` applies when `condition` holds on the frozen inputs. |
| `due` | Equal to the stage: the obligation is `expected` and resolves now. A later stage: it is listed in `future` as PLANNED. |
| `waiver_policy` | Family ceiling: `FORBIDDEN` is never waivable; `APPROVAL_REQUIRED` follows [gates Waivers](gates.md#waivers). |
| `reopen` | Earlier stages a FAIL, MAJOR finding or change found at this stage may reopen ([change-impact](change-impact.md)). |
| `check_order` | The fixed cost order of `run-checks`. |

`safety` and `required_by` are not used: safety comes from the requirement. `release.integrity` belongs to no stage
verdict; it is the post-approval package check inside `publish-release`.

## Lifecycle procedure IDs (owner cad-flow)

### resume-state

1. Confirm `.cad-work/lead/active/claim.yaml` names this session ([preflight](preflight.md)); otherwise stop.
2. Derive the chain from `hw/versions/v*/stage-verdict.json` and `hw/decisions/DR-*.yaml` ([cad-record's Versions](../../cad-record/references/record.md#versions)). The furthest approved stage is the last one whose PASS verdict has its gate satisfied and whose chain back to GROUND is unbroken and not reopened by a later CHANGE decision.
3. For its successor: a PASS verdict with an unsatisfied HUMAN gate → `present-gate`; an attempt without a
   verdict and with an unchanged input inventory digest → resume at its first unfinished step; a FAIL or BLOCKED
   verdict, or an unfinished attempt with changed inputs → a new attempt once the blocker is resolved.
4. `test -f` every path in `.cad-work/lead/preflight.json` → a missing path stops with `MISSING_RESOURCE`.

Output: the stage, its attempt `vN` or `new`, and the next procedure ID.

### freeze-inputs

1. Allocate `vN`: the next integer above every `hw/versions/v*`, `hw/checks/v*` and `.cad-work/stage-v*`;
   `mkdir .cad-work/stage-vN` must succeed. Never reuse a number.
2. Write `.cad-work/stage-vN/input-inventory.txt`: every record file, part model, check plan and case, retained
   source and accepted decision the attempt reads, with its digest ([cad-record's Inventory digest](../../cad-record/references/record.md#inventory-digest)).
3. From now until the verdict, a changed inventoried byte is `SOURCE_MUTATION`: stop.

### build-version

1. Recompute the input digest; a mismatch is `SOURCE_MUTATION`.
2. When the stage lists `geometry.build`, build and read back the integrated definition from the snapshot
   ([cad-model's model contract](../../cad-model/references/model-contract.md) and [exports](../../cad-model/references/exports.md)) → geometry `PRESENT`. Otherwise record
   geometry `NOT_APPLICABLE` with the reason: GROUND to VARIANT "no assembly definition yet", CRITIQUE
   "reviews the predecessor version", RELEASE "payload is the FREEZE version".
3. Stage the snapshot and geometry under `.cad-work/stage-vN/` and copy them following [cad-model's Version build contract](../../cad-model/references/exports.md#version-build), then write `manifest.json` (root `version-manifest`) last → the manifest validates.
4. An interrupted copy with unchanged inputs completes with identical bytes; changed inputs leave `vN`
   without a manifest, never resumed and never reused.

### enumerate-obligations

1. Evaluate every `checks[]` entry on the frozen inputs; record each false `condition` with its reason.
2. Take anchors from [cad-verify's check plan](../../cad-verify/references/check-plan.md). A family the check plan does not anchor uses `<family>@product`.
3. `due` equal to the stage → `expected`; a later `due` → `future`.
4. Write `.cad-work/stage-vN/obligations.yaml` (expected, future, inapplicable entries with reasons) before
   any check runs; the verdict's `expected` and `future` equal it.
5. An expected obligation with no check-plan method or no available tool is UNKNOWN `UNSUPPORTED_METHOD`; never
   substitute a method.

### run-checks

1. Run families in `check_order`. The lead runs `record.schema` ([cad-record validation](../../cad-record/references/validation.md)),
   `record.ownership`, `record.inputs` and `delivery.integrity` (hash and ownership audits). A `cad-checker`
   unit runs every other family; a `cad-mech-dfm-reviewer` unit runs `review.mechanical_dfm`. Units of families
   that read no result of another pending family go out in one batch ([delegation](delegation.md)).
2. Every stage result binds `VERSION` (vN and manifest digest). A worker's UNIT result is never stage evidence;
   a checker may read scratch first, then emits its VERSION result on the identical inventoried input.
3. When a batch has returned, accept its results ([worker-delivery](worker-delivery.md)) and publish each to
   `hw/checks/vN/<check>/<attempt>/` before the batch of any family that reads it is dispatched.
4. A FAIL or UNKNOWN in `record.*`, `delivery.integrity`, `geometry.build` or `geometry.readback` stops every
   later family; any other FAIL or UNKNOWN stops the families that read its result, artifacts or geometry.
   Each stopped obligation is UNKNOWN `NOT_RUN` naming the blocking obligation.
5. A rerun with unchanged inputs (interruption, timeout) is the next attempt `aN` of the same check with
   `supersedes`; a changed input, case or method is a new `vN`.
6. A result is `CARRIED_FORWARD` only under [change-impact Carry-forward](change-impact.md#carry-forward).
7. Fast or smoke runs stay in scratch as diagnostics and never enter a result.
8. A family whose subject a later procedure ID produces runs in that step ([cad-verify index-verification](../../cad-verify/references/check-plan.md#index-verification)).

### accept-results

1. Every expected obligation has one accepted outcome: PASS, NOT_APPLICABLE with its reviewer, the FAIL or UNKNOWN it ended with, or WAIVED when an APPROVED waiver `WV-###` covers that FAIL or UNKNOWN ([gates Waivers](gates.md#waivers)).
2. Every published result, bundle manifest and derived bundle validates against its root, and every recorded
   sha256 equals the bytes on disk.
3. Result scopes cover exactly `obligations.yaml`: nothing missing, nothing extra.

### write-verdict

1. Compose `hw/versions/vN/stage-verdict.json` ([cad-record stage-verdict schema](../../cad-record/schemas/stage-verdict.schema.json)): `predecessor` is the approved predecessor
   verdict and its approval; `supersedes` the attempt this one replaces; `expected`, `future`, `results`,
   `bundles`, `blockers`; `review.required` is true when the stage lists `review.mechanical_dfm` due now;
   `policy` is `{path, plugin}` from [preflight](preflight.md#preflightjson); predecessor approvals follow [cad-record decision bindings](../../cad-record/schemas/decision.schema.json#/$defs/binding); `time` is UTC.
2. Each `results[]` entry carries the check's result, except a FAIL or UNKNOWN check whose every non-PASS
   outcome an approved [cad-record waiver](../../cad-record/schemas/common.schema.json#/$defs/waiver) covers: its entry is `WAIVED` with that `waiver: WV-###`, and `waivers` lists
   `{id, decision}` for each waiver used.
3. `result`: PASS when complete, without blockers, every entry PASS, WAIVED or NOT_APPLICABLE and a required review PASS ([cad-review rubric](../../cad-review/references/rubric.md)). FAIL when a due obligation is evidenced FAIL or the required review is MINOR or MAJOR; BLOCKED otherwise.
4. `test ! -e` the target, validate against the linked cad-record stage-verdict schema, and write once → never rewritten.

### present-gate

HUMAN gates only: [gates](gates.md).

### advance

1. PASS verdict and satisfied gate → start the successor's procedure.
2. FAIL or BLOCKED → the stop procedure; the next attempt or reopen follows [change-impact](change-impact.md).
3. Batches never cross a stage whose gate is not satisfied.

### dispatch-units and merge-deliveries

`dispatch-units`: [delegation](delegation.md). `merge-deliveries`: [worker-delivery](worker-delivery.md). Zero units is
valid only when the stage work names none.

## Stage-work procedure IDs

Each owner reference defines the step under a heading with the procedure ID.

| ID | Owner reference | Output |
|---|---|---|
| `record-bootstrap` | [cad-record record-bootstrap](../../cad-record/references/record.md#record-bootstrap) | `hw/` from the template or an approved import |
| `intake-use-case` | [cad-select intake-use-case](../../cad-select/references/selection.md#intake-use-case) | intake, service, risk, authority, release scope, questions |
| `select-candidates` | [cad-select select-candidates](../../cad-select/references/selection.md#select-candidates) | candidates and the facts to source |
| `assess-selection` | [cad-select assess-selection](../../cad-select/references/selection.md#assess-selection) | `selection.yaml` assessment and recommendation |
| `specify-requirements` | [cad-verify specify-requirements](../../cad-verify/references/check-plan.md#specify-requirements) | requirement criteria, methods, facets, due stages |
| `specify-interfaces` | [cad-design specify-interfaces](../../cad-design/references/layout.md#specify-interfaces) | functional interface contracts |
| `plan-checks` | [cad-verify plan-checks](../../cad-verify/references/check-plan.md#plan-checks) | `check-plan.yaml` checks and cases |
| `draft-variants` | [cad-design draft-variants](../../cad-design/references/concepts.md#draft-variants) | 2–4 concept units and their merged variants |
| `compare-variants` | [cad-design compare-variants](../../cad-design/references/concepts.md#compare-variants) | comparison of the unchanged variants |
| `define-layout` | [cad-design define-layout](../../cad-design/references/layout.md#define-layout) | assembly tree, placements, joints, PN allocation, part units |
| `design-parts` | [cad-design design-parts](../../cad-design/references/parts.md#design-parts) | part and sourcing units per PN group |
| `detail-parts` | [cad-design detail-parts](../../cad-design/references/parts.md#detail-parts) | detail units: fits, tolerances, retention, process intent |
| `draft-drawings` | [cad-release draft-drawings](../../cad-release/references/drawings.md#draft-drawings) | drawing bundles for the version |
| `derive-bom` | [cad-release derive-bom](../../cad-release/references/bom.md#derive-bom) | BOM bundle for the version |
| `confirm-cases` | [cad-verify confirm-cases](../../cad-verify/references/check-plan.md#confirm-cases) | every due case approved with an available method |
| `index-verification` | [cad-verify index-verification](../../cad-verify/references/check-plan.md#index-verification) | verification bundle, then its `verification.index` check |
| `promote-revisions` | [cad-release promote-revisions](../../cad-release/references/release.md#promote-revisions) | promoted part revisions before final checks |
| `propose-release` | [cad-release propose-release](../../cad-release/references/release.md#propose-release) | `hw/versions/vN/release-proposal.json` |
| `publish-release` | [cad-release publish-release](../../cad-release/references/release.md#publish-release) | staged copy, `release.integrity`, final manifest |

## Stop procedure

Stop on any condition below; the code is the verdict blocker when an attempt has a manifest.

| Condition | Blocker |
|---|---|
| foreign lead claim, no authorized recovery, a missing skill, schema, role or tool | before a manifest: no verdict; else `MISSING_RESOURCE` |
| an installed file differs from what its contract states | `INSTALLED_MUTATION` |
| a due obligation with an UNKNOWN outcome; an UNKNOWN fact stops work only through the obligation where it falls due ([cad-select — Scope of each unknown](../../cad-select/references/selection.md#scope-of-each-unknown)) | `DUE_UNKNOWN` |
| a due FAIL | `DUE_FAIL` |
| missing result or incomplete producer | `MISSING_RESULT` |
| scope or coverage gap | `SCOPE_GAP` |
| schema-invalid file | `SCHEMA_INVALID` |
| hash or reference mismatch, package mismatch | `HASH_MISMATCH` |
| inventoried input changed during the attempt | `SOURCE_MUTATION` |
| change without its approval | `UNAPPROVED_CHANGE` |
| required review MAJOR | `REVIEW_MAJOR` |
| expired calibration or waiver | `EXPIRED` |
| timeout | `TIMEOUT` |
| rejected or failed worker delivery | `DELIVERY_FAILED` |

A missing human approval is no blocker: the PASS verdict waits `WAITING_FOR_USER`.

1. Dispatch nothing new; stop this session's running units that depend on the failed work.
2. Retain every failed result, log, delivery and scratch file; delete nothing.
3. An attempt with a manifest gets its FAIL or BLOCKED verdict (`write-verdict`).
4. Report with the SKILL.md OUTPUT shape: stage, vN, blocker code, obligation, absolute paths, and the
   decision or information that unblocks it.
5. Write nothing more until the user answers or the cause is resolved; then follow [change-impact](change-impact.md).
