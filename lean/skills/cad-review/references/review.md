# Mechanical and DFM review

Owner: cad-review. The read-only reviewer's procedure. cad-flow assigns cad-review no stage-work procedure ID:
[`review.mechanical_dfm`](../../cad-flow/references/stages.md#run-checks) is a check family dispatched to a
`cad-mech-dfm-reviewer` unit at PARTS, CRITIQUE and FREEZE. Grading is [cad-review rubric](rubric.md#verdict);
the reply is [cad-flow agent roles — Reply](../../cad-flow/references/agent-roles.md#reply); the lead's transcription
is [cad-flow worker delivery — Read-only reviewer](../../cad-flow/references/worker-delivery.md#read-only-reviewer).

## review.mechanical_dfm

Input: the brief's `version`, `subjects`, `expected.checks` and `expected.obligations`, and `input_inventory` (each
file with its sha256): requirements, interfaces, assembly, parts, the version manifest, published results with their
bundle manifests and `source/` programs, obligations, drawings, BOM and verification index.

1. Read only the copies under `.cad-work/<unit>/input/` and the brief's `boundary.read_only` paths. Run, hash,
   validate and write nothing. A file you need that is not in the inventory is a MAJOR finding, never a guess.
2. Record each file you read, other checks' `result.json` and `bundle-manifest.json` included, as an `inspected`
   entry with the sha256 the brief inventory gives for it; never list this review's own result.
3. List coverage from the version's expected obligations, one entry per obligation:
   - `mechanical`: `requirements.completeness`, `interfaces.completeness`, `verification.index`,
     `geometry.interference`, `geometry.clearance`, `motion.*`, `assembly.operations`, `fit.*`, `mass.properties`,
     `load.*`, `physical.test`;
   - `dfm`: `material.properties`, `selection.sourcing`, `purchase.identity`, `geometry.manufacture`,
     `capability.route`, `optical.route`, `process.*`, `drawing.coverage`, `drawing.preview`, `bom.derivation`.
   An empty list holds one entry `review.mechanical_dfm@assembly`, NOT_APPLICABLE with its reason.
4. For each entry decide the outcome the evidence supports: PASS only when its result is PASS and the challenges below
   find no defect; FAIL, UNKNOWN, NOT_APPLICABLE or WAIVED (an APPROVED waiver DR in the inventory) with a `reason`.
5. Challenge the premise: each requirement traces use case → requirement → criterion → check; criteria and facets
   equal the approved SPECIFY record and the service class ([cad-verify check plan](../../cad-verify/references/check-plan.md)); nothing is weakened.
6. Challenge the evidence of each result:
   - its scope equals its obligations, and no pair, pose, component or feature was dropped;
   - coverage FULL, or SAMPLED with unsampled intervals listed;
   - its `source/` program measures what the outcome claims (pairs, poses, contributors, criteria) on inputs that
     hash to this version;
   - no PHYSICALLY_VALIDATED claim without a physical test ([cad-verify results — Physical evidence](../../cad-verify/references/results.md#physical-evidence));
   - every CARRIED_FORWARD result names its recheck.
7. Mechanical: the load path from each load to its reaction; retention, fastener access, engagement and edge
   distance; joints over full travel and at play extremes; fits under the product regime; stacks at the worst case;
   stiffness, stability, thermal and moisture effects; FEA validity (boundary conditions, mesh, convergence,
   singularities, units).
8. DFM: each part and operation by its own route kind against [cad-verify process checks](../../cad-verify/references/process-checks.md), never one
   process for the whole product; supports on critical surfaces, tool access, workholding, stock identity, kerf and
   thickness ranges, purchased identity; drawings carry every characteristic the route and inspection need; the BOM
   is derived, not typed. A process without criteria or capability is untested: UNKNOWN, never PASS.
9. Write findings with the fields of [cad-verify results — Findings](../../cad-verify/references/results.md#findings): each measured value with units and
   the inspected path it came from; remedy with alternatives and tradeoffs; `recheck` naming the families and the
   earliest causal stage ([cad-flow change impact — Findings](../../cad-flow/references/change-impact.md#findings)). Remedies are proposals; you redesign nothing.
10. Grade the version with [cad-review rubric — Verdict](rubric.md#verdict) → `PASS`, `MINOR` or `MAJOR`.
11. Compose the result document (root `check-result`) and reply with it as one fenced `json` block.

## Result document

| Field | Value |
|---|---|
| `format`, `schema_version` | `cad-agent-skills.check-result`, `1` |
| `id`, `check`, `attempt` | `<check>/<attempt>` from the brief's `expected.checks` |
| `family`, `producer_role`, `stage` | `review.mechanical_dfm`, `cad-mech-dfm-reviewer`, the brief's stage |
| `binding` | `{kind: VERSION, version, version_manifest_sha256}`: the manifest sha256 from the brief inventory |
| `input_files`, `implementation` | the inspected list; `{kind: REVIEW, inspected}` with no commands |
| `scope` | subjects `[{kind: ASSEMBLY}]`, `expected_obligations: [review.mechanical_dfm@assembly]`, `coverage_kind` FULL when every coverage entry was assessed, else PARTIAL |
| `outcomes` | one: `review.mechanical_dfm@assembly`, the check-plan criterion and facet, value `{text: "review verdict <VERDICT>"}`, outcome PASS for a PASS verdict and FAIL otherwise, evidence the inspected result files |
| `findings`, `artifacts` | every finding; `[]` (the reviewer writes no artifact) |
| `complete`, `result` | true when every coverage entry was assessed; PASS for a PASS verdict, else FAIL |
| `limitations` | untested checks, unread evidence, sampled scope, absent physical evidence |
| `review` | `{verdict, mechanical, dfm}` from steps 3–4 and 10 |
The lead saves the block byte-identically and validates it; a block that fails validation, lists commands or
mismatches the inventory is rejected and re-dispatched as a new unit, never repaired.
