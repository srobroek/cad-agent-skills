# Contract index

Owner: cad-record (U1). Lookup for every other skill and role. The schemas under `schemas/` are the field
authority; references/record.md owns the rules behind them. `<cad-record>` and `<cad-flow>` are absolute skill directories.

## Schema roots

Validate with `check-jsonschema --schemafile <cad-record>/schemas/<root>.schema.json <instance>` (references/validation.md).

| Root | Contract | Instances |
|---|---|---|
| `common` | IDs, paths, hashes, units, facts, sources, criteria, subjects, obligations, findings, waivers, authority, binding | none (definitions; root validates contract probes only) |
| `product` | namespace, record version, identity, frame, rigor, regime, intake, service, risk, authority, release scope, storage, license | `hw/product.yaml` |
| `requirements` | requirements, preferences, assumptions, questions | `hw/requirements.yaml` |
| `interfaces` | functional interface contracts | `hw/interfaces.yaml` |
| `part` | one part: sourcing, model, components, manufacture, intent, procurement, drawing | `hw/parts/{PN}/part.yaml` |
| `assembly` | occurrence tree, joints, couplings, operations, service | `hw/assembly.yaml` |
| `bom-row` | one derived BOM row | items of `bom.rows` |
| `bom` | derived BOM for vN | `hw/bom/{V}/bom.yaml`, `hw/releases/{REV}/bom.yaml` |
| `verification-row` | one derived requirement x subject x facet row | items of `verification.rows` |
| `verification` | derived verification index for vN | `hw/verification/{V}/verification.yaml`, `hw/releases/{REV}/verification.yaml` |
| `decision` | one immutable DR | `hw/decisions/{DR}.yaml` |
| `check-result` | one check attempt result (sole observation authority) | `hw/checks/{V}/{CHECK}/{ATTEMPT}/result.json`, `hw/drawings/{V}/{SUBJECT}/coverage.json`, `hw/releases/{REV}/acceptance/package-integrity.json` |
| `stage-verdict` | one immutable stage verdict | `hw/versions/{V}/stage-verdict.json` |
| `stage-policy` | ordered stage, gate and check policy | `<cad-flow>/assets/stages.yaml` |
| `material-profile` | one material profile | items of a material library `profiles` |
| `material-library` | material profiles wrapper | `hw/manufacturing/materials.yaml`, `<cad-record>/assets/manufacturing/material-profiles/*.yaml` |
| `capability-profile` | one capability profile or template | items of a capability library `profiles` |
| `capability-library` | capability profiles wrapper | `hw/manufacturing/capabilities.yaml`, `<cad-record>/assets/manufacturing/capability-templates.yaml` |
| `selection` | material, route and architecture candidates | `hw/manufacturing/selection.yaml` |
| `check-plan` | planned checks and typed cases | `hw/check-plan.yaml` |
| `variant` | one concept variant | `hw/concepts/{VARIANT}/variant.yaml` |
| `version-manifest` | input and geometry inventory of vN | `hw/versions/{V}/manifest.json` |
| `bundle-manifest` | bundle inventory, excludes itself | `hw/checks/{V}/{CHECK}/{ATTEMPT}/bundle-manifest.json`, `hw/drawings/{V}/{SUBJECT}/bundle-manifest.json`, `hw/bom/{V}/bundle-manifest.json`, `hw/verification/{V}/bundle-manifest.json` |
| `worker-brief` | typed worker assignment | `.cad-work/{UNIT}/brief.yaml` |
| `worker-delivery` | typed worker return | `.cad-work/{UNIT}/delivery.json` |
| `lead-claim` | the one live lead | `.cad-work/lead/active/claim.yaml` |
| `release-manifest` | PROPOSAL or FINAL release manifest | `hw/versions/{V}/release-proposal.json`, `hw/releases/{REV}/release-manifest.json` |
| `layout` | runtime path catalog | `<cad-record>/layout.json` |

## Assets

- Template: `<cad-record>/assets/project-template/hw/` (product, requirements, interfaces, assembly, check-plan,
  manufacturing/{materials,capabilities,selection}); open questions `Q-001`…`Q-017`.
- Contract fixtures: `<cad-record>/assets/contract-fixtures/<root>/{valid,invalid}/<case>.{yaml,json}`; the
  intended error of each invalid case is in `contract-fixtures/expected-errors.yaml`.
- Authoring schemas (Draft-07, repository `full/schemas/authoring/`): `skill-frontmatter`, `omp-agent-frontmatter`,
  `claude-agent-frontmatter`, `opencode-agent-frontmatter`, `codex-agent`, `kiro-agent`, `claude-plugin`,
  `omp-plugin`, `claude-marketplace`, `omp-marketplace`; fixtures under `full/schemas/authoring/fixtures/`.

## ID families

| Family | Pattern (common `$defs`) |
|---|---|
| `pn` | `^[A-Z]{2,5}-[0-9]{4}$` |
| `part_rev` | `^(X[1-9][0-9]*\|[A-HJ-NPRTUVWY])$` |
| `release_rev` | `^[A-HJ-NPRTUVWY]$` |
| `requirement_id` | `^REQ-[0-9]{3}$` |
| `interface_id` | `^IF-[0-9]{3}$` |
| `decision_id` | `^DR-[0-9]{3}$` |
| `question_id` | `^Q-[0-9]{3}$` |
| `assumption_id` | `^ASM-[0-9]{3}$` |
| `preference_id` | `^PRF-[0-9]{3}$` |
| `waiver_id` | `^WV-[0-9]{3}$` (`waiver.id` inside a WAIVER DR) |
| `version` | `^v[1-9][0-9]*$` |
| `attempt` | `^a[1-9][0-9]*$` |
| `slug` | `^[a-z][a-z0-9_-]*$` |
| `component_id` | `^[a-z][a-z0-9_]*$` |
| `semantic_id` | `^[a-z][a-z0-9_]*$` |
| `profile_id` | `^[a-z][a-z0-9_]*$` |
| `unit_id` | `^(?!lead$)(?!stage-)[a-z][a-z0-9_-]*$` |
| `product_id` | `^[a-z0-9][a-z0-9-]*$` |
| `sha256` | `^[0-9a-f]{64}$` |
| `date` | `^[0-9]{4}-(0[1-9]\|1[0-2])-(0[1-9]\|[12][0-9]\|3[01])$` |
| `timestamp` | `^[0-9]{4}-(0[1-9]\|1[0-2])-(0[1-9]\|[12][0-9]\|3[01])T([01][0-9]\|2[0-3]):[0-5][0-9]:[0-5][0-9](\.[0-9]+)?Z$` |
| `obligation_id` | `<family>@<anchor>` |
| `finding_id` | `<family>:<anchor>:<feature-or-operation>` |
| check result `id` | `<check>/<attempt>` |

Anchor: `product`, `assembly`, `<PN>[.<component>][/<operation>][#<feature>]`, `IF-###[#<feature>]`, `REQ-###`,
`op.<assembly operation>` or `case.<case>`. Families are the registered list below.

## Registered check families

`common.schema.json#/$defs/check_family`; stage policy, check plan, results, findings and obligation IDs use only these.

- record: `record.schema`, `record.semantic`, `record.ownership`, `record.inputs`
- delivery: `delivery.integrity`
- intake: `intake.use_case`
- selection: `selection.feasibility`, `selection.sourcing`
- material: `material.properties`
- requirements: `requirements.completeness`
- interfaces: `interfaces.completeness`
- check_plan: `check_plan.completeness`
- variant: `variant.integrity`
- geometry: `geometry.build`, `geometry.readback`, `geometry.export`, `geometry.identity`, `geometry.manufacture`,
  `geometry.interference`, `geometry.clearance`
- motion: `motion.sweep`, `motion.play`, `motion.access`
- assembly: `assembly.operations`
- fit: `fit.limits`, `fit.stack`
- process: `process.fdm`, `process.resin`, `process.cnc`, `process.laser`, `process.waterjet`, `process.plasma`,
  `process.sheet_forming`, `process.joining`, `process.casting`, `process.moulding`, `process.extrusion`,
  `process.secondary`, `process.other`
- capability: `capability.route`
- purchase: `purchase.identity`
- optical: `optical.route`
- mass: `mass.properties`
- load: `load.hand`, `load.fea`
- drawing: `drawing.coverage`, `drawing.preview`
- preview: `preview.render`
- bom: `bom.derivation`
- verification: `verification.index`
- review: `review.mechanical_dfm`
- physical: `physical.test`
- release: `release.inventory`, `release.integrity`

## Enumerations

- Stages, in order: `GROUND`, `INTENT`, `SPECIFY`, `CONCEPT`, `VARIANT`, `LAYOUT`, `PARTS`, `DETAIL`, `BOM`, `ANALYSIS`,
  `CRITIQUE`, `FREEZE`, `RELEASE`.
- Facets: `CAD_VALID`, `GEOMETRICALLY_MANUFACTURABLE`, `PROCESS_PLANNED`, `SIMULATION_SUPPORTED`, `PHYSICALLY_VALIDATED`.
- Obligation outcome: `PASS|FAIL|UNKNOWN|NOT_APPLICABLE|WAIVED`; verification row adds `PLANNED`.
- Check `result`: `PASS|FAIL|UNKNOWN|NOT_APPLICABLE`; stage verdict `result`: `PASS|FAIL|BLOCKED`.
- Review and worker verdict: `PASS|MINOR|MAJOR`. Severity: `CRITICAL|MAJOR|MINOR|INFO`.
- `unknown_code`: `NOT_RUN`, `INCOMPLETE`, `STALE`, `MISSING_INPUT`, `UNSUPPORTED_METHOD`, `TIMEOUT`,
  `REQUIRED_UNKNOWN`, `PHYSICAL_EVIDENCE_REQUIRED`.
- Service class: `APPEARANCE_CONCEPT`, `FIT_ASSEMBLY`, `FUNCTIONAL_DEMO`, `WORKING_PROTOTYPE`, `PRODUCTION`,
  `SAFETY_CRITICAL`, `UNKNOWN`. Rigor: `maker|pro|regulated`. Kernel: `cadquery|build123d`.
- Sourcing: `MAKE|BUY|MADE_TO_DRAWING`. Primary process: `FDM`, `RESIN`, `CNC`, `LASER`, `WATERJET`, `PLASMA`,
  `SHEET_FORMING`, `CASTING`, `MOULDING`, `EXTRUSION`, `PURCHASED`, `OTHER`.
- Route kind: `FDM`, `RESIN`, `CNC`, `LASER`, `WATERJET`, `PLASMA`, `SHEET_FORMING`, `JOINING`, `CASTING`, `MOULDING`,
  `EXTRUSION`, `DRILLING`, `REAMING`, `TAPPING`, `FINISHING`, `COATING`, `HEAT_TREATMENT`, `INSPECTION`, `PURCHASED`,
  `OTHER`.
- Fact status: `KNOWN|UNKNOWN|NOT_APPLICABLE`; basis:
  `MEASURED|SUPPLIER_TYPICAL|STANDARD|USER_DECLARED|CALCULATED|ESTIMATE|ASSUMED|ALLOWABLE`.
- Units: `mm`, `um`, `m`, `mm2`, `mm3`, `deg`, `rad`, `N`, `kN`, `N_mm`, `N_m`, `N_per_mm`, `Pa`, `kPa`, `MPa`, `GPa`,
  `bar`, `g`, `kg`, `kg_m3`, `g_cm3`, `deg_C`, `K`, `s`, `min`, `h`, `d`, `cycles`, `count`, `rpm`, `Hz`, `mm_s`, `m_s`,
  `deg_s`, `percent`, `dimensionless`, `J`, `J_m`, `kJ_m2`, `W`, `kW`, `W_m_K`, `um_m_K`, `per_K`, `V`, `A`.
- Producer roles: `cad-concept-designer`, `cad-part-designer`, `cad-checker`, `cad-mech-dfm-reviewer`,
  `cad-sourcing-researcher`, `lead`. Skills: `cad-flow`, `cad-record`, `cad-select`, `cad-design`, `cad-model`,
  `cad-verify`, `cad-review`, `cad-release`.

## Check result envelope (`check-result`)

| Field | Contract |
|---|---|
| `format`, `schema_version` | `cad-agent-skills.check-result`, `1` |
| `id`, `check`, `attempt`, `family` | `<check>/<attempt>`, check-plan id, `aN`, registered family |
| `producer_role`, `stage` | role or `lead`; stage enum |
| `binding` | `{kind: UNIT, unit, brief_sha256, input_inventory_sha256}` or `{kind: VERSION, version, version_manifest_sha256}` |
| `input_files` | `[{path, sha256}]` of every input read |
| `implementation` | `kind: COMMAND` with `{source_files, commands[{argv, cwd, exit_code}] (at least one), packages, platform}`, or `kind: REVIEW` with `{inspected[{path, sha256}] (at least one)}` and no commands; the reviewer always reports REVIEW |
| `scope` | `{subjects, components, operations, cases, expected_obligations, coverage_kind FULL, SAMPLED or PARTIAL, unsampled}` |
| `outcomes[]` | `{obligation, requirement, subject, facet, outcome, value, criterion, method, evidence, sampling, …}` |
| `findings[]` | `{id, family, subject, severity, outcome, measured, criterion, method, uncertainty, consequence, remedy, alternatives, recheck}` |
| `artifacts[]` | `{path, sha256, role}`; never `result.json` or `bundle-manifest.json` |
| `complete`, `result`, `limitations` | PASS needs complete, FULL or SAMPLED coverage, no FAIL/UNKNOWN/WAIVED outcome |
| `review` | reviewer only: `{verdict, mechanical[], dfm[]}; verdict PASS, MINOR or MAJOR` coverage of obligations |
| `release` | `release.integrity` only: `{rev, approval, planned_inventory}` |
| `supersedes` | `{attempt, reason}` for a retry |

Outcome rules: PASS needs `value`, a real `criterion` and `evidence`; FAIL the same; UNKNOWN needs `reason` and
`unknown_code`; NOT_APPLICABLE needs `reason` and `reviewed_by`; WAIVED needs the waiver DR and its `limitation`.

Result files outside `hw/checks/`: drawing `coverage.json` uses family `drawing.coverage`, check
`drawing-<subject in lowercase>` and attempt `a1`; `acceptance/package-integrity.json` uses family
`release.integrity`, check `package-integrity`, attempt `a1` and VERSION binding. A redo is a new vN or REV.
The `common` root validates contract probes only (`probe: cad-agent-skills.common`); no runtime file uses it.

## Stage verdict (`stage-verdict`)

`stage`, `version`, `predecessor{stage, version, verdict, approval}` (required after GROUND), `supersedes`,
`version_manifest{path, sha256}`, `policy{path (absolute), plugin{name, version}}`, `expected[obligation_id]`,
`results[{check, attempt, family, result, waiver, binding VERSION|CARRIED_FORWARD, result_file, bundle, carried_from}]`,
`bundles[{kind DRAWING|BOM|VERIFICATION, manifest}]`, `waivers[{id WV-###, decision DR-###}]`,
`future[obligation_id]` (PLANNED), `blockers[{code, text, obligation}]`, `review{required, verdict, result}`,
`complete`, `result`, `time`.

`results[].result` is the stage mapping `PASS|FAIL|UNKNOWN|NOT_APPLICABLE|WAIVED`; `WAIVED` maps a FAIL or UNKNOWN
check covered by a permitted waiver and requires `waiver` (WV-###); `waiver` is forbidden on every other result.
The check's own `result.json` keeps its FAIL or UNKNOWN. PASS needs `complete`, no blockers, only PASS, WAIVED and
NOT_APPLICABLE results, and a PASS review when one is required. Waiver existence, scope and expiry are semantic
checks (references/validation.md).

## Bundle manifest, brief and delivery

- `bundle-manifest`: `kind CHECK|DRAWING|BOM|VERIFICATION`, `binding`, `input[{path, sha256}]`, `recipe` (`kind: COMMAND`
  `{commands, packages, platform}` or `kind: REVIEW` `{inspected}`), `policy{path, plugin}`, `files[{path, sha256, role}]`
  without itself, `complete`.
- `worker-brief`: `resources{plugin{name, version}, skills[{name, path, uri}], schemas[{name, path}],
  role_reference{name, path, uri}}` with absolute paths and no hashes of installed files; `input_inventory{sha256,
  files[{path under hw/, sha256}]}`.
- `worker-delivery`: `brief_sha256`, `input_inventory_sha256`, `boundary{method, before_sha256, after_sha256, violations}`.
- Every inventory and boundary digest follows references/record.md#inventory-digest. Installed skills, schemas and
  policy are identified by `plugin{name, version}` and their resolved absolute paths; nothing installed is hashed.
  On every host the version source is the installed cad-flow SKILL.md frontmatter
  `metadata: {plugin: cad-agent-skills, version: "<semver>"}` (lean: `cad-agent-skills-lean`); the
  `skill-frontmatter` authoring schema requires both keys on cad-flow.

## Stage policy (`stage-policy`, written by cad-flow)

`stages[13]{id, order, predecessor, gate HUMAN|NONE|REVIEW, gate_subject, owner (skill), procedure[ids],
checks[{family, applicability ALWAYS|CONDITIONAL, condition, due, required_by, safety, waiver_policy}], reopen[stage]}`,
and `check_order[family]` (the fixed cost order).

## Decision bindings (`decision`)

`binding.kind`: `PROPOSAL{path under hw/decisions/proposals/DR-###/, sha256}`, `STAGE{stage, version, verdict, verdict_result}`,
`FROZEN{version, version_manifest_sha256, verdict}`, `RELEASE_INVENTORY{rev, version, proposal}`. A GATE DR binds STAGE;
an APPROVED gate binds a PASS verdict; APPROVED and REJECTED need a USER or AUTHORITY actor and a transcript statement.

## Paths

`layout.json` entry ids other skills cite (path templates use its tokens):

- `record.product`: `hw/product.yaml` → `product`
- `record.requirements`: `hw/requirements.yaml` → `requirements`
- `record.interfaces`: `hw/interfaces.yaml` → `interfaces`
- `record.assembly`: `hw/assembly.yaml` → `assembly`
- `record.check_plan`: `hw/check-plan.yaml` → `check-plan`
- `record.materials`: `hw/manufacturing/materials.yaml` → `material-library`
- `record.capabilities`: `hw/manufacturing/capabilities.yaml` → `capability-library`
- `record.selection`: `hw/manufacturing/selection.yaml` → `selection`
- `record.manufacturing_source`: `hw/manufacturing/sources/{SHA}/{NAME}`
- `record.part`: `hw/parts/{PN}/part.yaml` → `part`
- `record.part_model`: `hw/parts/{PN}/model.py`
- `record.part_source`: `hw/parts/{PN}/sources/**`
- `record.variant`: `hw/concepts/{VARIANT}/variant.yaml` → `variant`
- `record.variant_architecture`: `hw/concepts/{VARIANT}/architecture.md`
- `record.variant_geometry`: `hw/concepts/{VARIANT}/geometry/{NAME}`
- `record.decision`: `hw/decisions/{DR}.yaml` → `decision`
- `record.decision_proposal`: `hw/decisions/proposals/{DR}/{NAME}`
- `notes.docs`: `hw/docs/**`
- `version.record`: `hw/versions/{V}/record/**`
- `version.source`: `hw/versions/{V}/source/**`
- `version.manifest`: `hw/versions/{V}/manifest.json` → `version-manifest`
- `version.stage_verdict`: `hw/versions/{V}/stage-verdict.json` → `stage-verdict`
- `version.release_proposal`: `hw/versions/{V}/release-proposal.json` → `release-manifest`
- `version.definition_step`: `hw/versions/{V}/definition/assembly.step`
- `version.definition_glb`: `hw/versions/{V}/definition/assembly.glb`
- `version.definition_part`: `hw/versions/{V}/definition/parts/{PN}.step`
- `version.manufacturing_step`: `hw/versions/{V}/manufacturing/{PN}/part.step`
- `version.manufacturing_stl`: `hw/versions/{V}/manufacturing/{PN}/part.stl`
- `version.manufacturing_summary`: `hw/versions/{V}/manufacturing/{PN}/manufacturing.yaml`
- `version.manufacturing_flat_dxf`: `hw/versions/{V}/manufacturing/{PN}/flat.dxf`
- `version.manufacturing_flat_svg`: `hw/versions/{V}/manufacturing/{PN}/flat.svg`
- `version.component_step`: `hw/versions/{V}/manufacturing/{PN}/components/{COMP}/part.step`
- `version.component_stl`: `hw/versions/{V}/manufacturing/{PN}/components/{COMP}/part.stl`
- `version.preview_assembly`: `hw/versions/{V}/previews/assembly.svg`
- `version.preview_part`: `hw/versions/{V}/previews/parts/{PN}.svg`
- `check.result`: `hw/checks/{V}/{CHECK}/{ATTEMPT}/result.json` → `check-result`
- `check.bundle_manifest`: `hw/checks/{V}/{CHECK}/{ATTEMPT}/bundle-manifest.json` → `bundle-manifest`
- `check.source`: `hw/checks/{V}/{CHECK}/{ATTEMPT}/source/**`
- `check.artifacts`: `hw/checks/{V}/{CHECK}/{ATTEMPT}/artifacts/**`
- `check.stdout`: `hw/checks/{V}/{CHECK}/{ATTEMPT}/stdout.txt`
- `check.stderr`: `hw/checks/{V}/{CHECK}/{ATTEMPT}/stderr.txt`
- `drawing.sheet_svg`: `hw/drawings/{V}/{SUBJECT}/sheet-{N}.svg`
- `drawing.sheet_pdf`: `hw/drawings/{V}/{SUBJECT}/sheet-{N}.pdf`
- `drawing.sheet_png`: `hw/drawings/{V}/{SUBJECT}/sheet-{N}.png`
- `drawing.coverage`: `hw/drawings/{V}/{SUBJECT}/coverage.json` → `check-result`
- `drawing.bundle_manifest`: `hw/drawings/{V}/{SUBJECT}/bundle-manifest.json` → `bundle-manifest`
- `bom.yaml`: `hw/bom/{V}/bom.yaml` → `bom`
- `bom.csv`: `hw/bom/{V}/bom.csv`
- `bom.bundle_manifest`: `hw/bom/{V}/bundle-manifest.json` → `bundle-manifest`
- `verification.yaml`: `hw/verification/{V}/verification.yaml` → `verification`
- `verification.csv`: `hw/verification/{V}/verification.csv`
- `verification.bundle_manifest`: `hw/verification/{V}/bundle-manifest.json` → `bundle-manifest`
- `release.manifest`: `hw/releases/{REV}/release-manifest.json` → `release-manifest`
- `release.integrity`: `hw/releases/{REV}/acceptance/package-integrity.json` → `check-result`
- `release.definition`: `hw/releases/{REV}/definition/**`
- `release.manufacturing`: `hw/releases/{REV}/manufacturing/**`
- `release.drawings`: `hw/releases/{REV}/drawings/**`
- `release.record`: `hw/releases/{REV}/record/**`
- `release.checks`: `hw/releases/{REV}/checks/**`
- `release.docs`: `hw/releases/{REV}/docs/**`
- `release.bom_yaml`: `hw/releases/{REV}/bom.yaml` → `bom`
- `release.bom_csv`: `hw/releases/{REV}/bom.csv`
- `release.verification_yaml`: `hw/releases/{REV}/verification.yaml` → `verification`
- `release.verification_csv`: `hw/releases/{REV}/verification.csv`
- `scratch.lead_preflight`: `.cad-work/lead/preflight.json`
- `scratch.lead_units`: `.cad-work/lead/units.yaml`
- `scratch.lead_sessions`: `.cad-work/lead/sessions/**`
- `scratch.lead_claim`: `.cad-work/lead/active/claim.yaml` → `lead-claim`
- `scratch.unit_brief`: `.cad-work/{UNIT}/brief.yaml` → `worker-brief`
- `scratch.unit_input`: `.cad-work/{UNIT}/input/**`
- `scratch.unit_output`: `.cad-work/{UNIT}/output/**`
- `scratch.unit_build`: `.cad-work/{UNIT}/build.py`
- `scratch.unit_check`: `.cad-work/{UNIT}/check.py`
- `scratch.unit_delivery`: `.cad-work/{UNIT}/delivery.json` → `worker-delivery`
- `scratch.stage`: `.cad-work/stage-{V}/**`
