# Prose contract

Owner: cad-record. This variant ships `common.schema.json` and ten schema roots: `product`, `requirements`, `part`,
`assembly`, `decision`, `check-result`, `stage-verdict`, `material-profile`, `bom-row` and `release-manifest`. The 17
roots below have no schema file. Each table here is the field authority for its root and has the same shape as the
full variant's schema of that name, so a conforming file also validates there. `record.semantic` item 17
(references/validation.md) checks every instance against its table; no schema command runs on a prose root.

| Prose root | Instances (layout id) |
| --- | --- |
| [interfaces](#interfaces) | `hw/interfaces.yaml` (`record.interfaces`) |
| [check-plan](#check-plan) | `hw/check-plan.yaml` (`record.check_plan`) |
| [material-library](#material-library) | `hw/manufacturing/materials.yaml` (`record.materials`), `<cad-record>/assets/manufacturing/material-profiles/*.yaml` |
| [capability-profile](#capability-profile) | items of a capability library `profiles` |
| [capability-library](#capability-library) | `hw/manufacturing/capabilities.yaml` (`record.capabilities`), `<cad-record>/assets/manufacturing/capability-templates.yaml` |
| [selection](#selection) | `hw/manufacturing/selection.yaml` (`record.selection`) |
| [variant](#variant) | `hw/concepts/{VARIANT}/variant.yaml` (`record.variant`) |
| [version-manifest](#version-manifest) | `hw/versions/{V}/manifest.json` (`version.manifest`) |
| [bundle-manifest](#bundle-manifest) | every `bundle-manifest.json` (`check.`, `drawing.`, `bom.` and `verification.bundle_manifest`) |
| [bom](#bom) | `hw/bom/{V}/bom.yaml` (`bom.yaml`), `hw/releases/{REV}/bom.yaml` (`release.bom_yaml`) |
| [verification](#verification) | `hw/verification/{V}/verification.yaml` (`verification.yaml`), `hw/releases/{REV}/verification.yaml` (`release.verification_yaml`) |
| [verification-row](#verification-row) | items of `verification.rows` |
| [stage-policy](#stage-policy) | `<cad-flow>/assets/stages.yaml` (installed, read-only) |
| [worker-brief](#worker-brief) | `.cad-work/{UNIT}/brief.yaml` (`scratch.unit_brief`) |
| [worker-delivery](#worker-delivery) | `.cad-work/{UNIT}/delivery.json` (`scratch.unit_delivery`) |
| [lead-claim](#lead-claim) | `.cad-work/lead/active/claim.yaml` (`scratch.lead_claim`) |
| [layout](#layout) | `<cad-record>/layout.json` (installed, read-only) |

A snapshot under `hw/versions/vN/record/` conforms to the prose root of the live file it copies.

## Reading a table

- Every object is closed: a field that neither its table nor its type lists is a defect.
- Req `yes` is required and `—` optional. `when X` is required when X holds and allowed otherwise; `iff X` is
  required when X holds and forbidden otherwise; `never …` forbids the field in the named case.
- A bare type name (`text`, `sha256`, `quantity_fact`, `fact_open`…) is that entry of `common.schema.json` `$defs`;
  the value validates against it, including its own conditions. `part#purchase` is `part.schema.json`
  `$defs/purchase`. `root:<name>` is a lean schema root; the value validates against that root file.
- `A|B` is a closed CAPS enum; `=X` a constant; `{a, b}` a closed object whose fields are listed;
  `[T]` an array of T of any length, `[T]≥1` at least one item, `unique` no repeated item.
- `fact_open` is an explicit UNKNOWN or NOT_APPLICABLE fact and `fact_unknown` an explicit UNKNOWN; neither is
  ever an omission, null, zero or default.

## Normative rules

These rules bind the prose roots exactly as they bind the schema roots.

- UNKNOWN never becomes PASS: a verification row copies its outcome from check results; a hard outcome UNKNOWN
  keeps a candidate or variant out of FEASIBLE; a FAIL or UNKNOWN check keeps a delivery from a ready PASS.
- Release ceiling: no prose-root field records physical validation or a functional pass that a physical test result
  does not carry ([cad-verify physical evidence](../../cad-verify/references/results.md#physical-evidence),
  [cad-release](../../cad-release/references/release.md)).
- Transparency: a selection or variant never meets an optical need with an FDM route; the rule is cad-select's
  ([cad-select](../../cad-select/SKILL.md#rules)) and the `optical.route` check verifies it.
- Hash graph: every digest field below follows [record.md](record.md#acyclic-hash-graph) and
  [Inventory digest](record.md#inventory-digest); no file holds its own digest or a downstream digest.
- Installed skills, schemas, policy and this file are read-only; a mismatch is a defect and an open question.

## interfaces

Authored record YAML, no header. Top level: `interfaces` (yes, `[interface]`). Every feature is a semantic name,
never a face index.

| Field | Req | Type and rule |
| --- | --- | --- |
| `id` | yes | `interface_id` |
| `title` | yes | `text` |
| `kind` | yes | `FIXED\|SLIDING\|ROTATING\|PRESS_FIT\|FASTENED\|SNAP\|CONTACT\|CLEARANCE\|SEAL\|ELECTRICAL\|OTHER` |
| `members` | yes | `[member]≥2`; member `{PN pn, feature semantic_id, role}` required, `occurrence semantic_id`, `component component_id` optional; role `BORE\|SHAFT\|SEAT\|FACE\|SLOT\|PIN\|THREAD\|SNAP\|SURFACE\|OTHER` |
| `functions` | yes | `[function_entry]≥1` or `fact_unknown` |
| `critical` | yes | `boolean_fact` |
| `requirements` | — | `requirement_ids` |
| `datums` | — | `[{id semantic_id, PN pn, feature semantic_id, order PRIMARY\|SECONDARY\|TERTIARY}]≥1`, all required |
| `fit` | yes | `{kind CLEARANCE\|TRANSITION\|INTERFERENCE, designation text_fact, state dimension_state, source source}` required, `hole`, `shaft` `quantity_fact` optional; or `fact_open` |
| `clearance` | yes | `{minimum quantity_fact}` required, `maximum quantity_fact`, `criterion criterion` optional; or `fact_open` |
| `play` | yes | `{axial, radial, angular}` `quantity_fact` required, `backlash quantity_fact` and `cases [case_id]≥1 unique` optional; or `fact_open` |
| `variation` | yes | `{stack_case case_id, method WORST_CASE\|JUSTIFIED_RSS}` required, `decision decision_id` when `JUSTIFIED_RSS`; or `fact_open` |
| `contact` | yes | `{materials, finish text_fact, lubrication text_fact, load quantity_fact, speed quantity_fact, duty text_fact, life quantity_fact}` all required, `materials` exactly two items each `profile_id` or `fact_unknown`; or `fact_open` |
| `environment` | yes | `text_fact` |
| `access` | yes | `{tool, install, service, inspection}`, each `text_fact`, all required |
| `assembly_operations` | — | `[slug]≥1 unique` |
| `acceptance` | — | `[criterion]≥1` |
| `decisions` | — | `decision_ids` |

## check-plan

Authored record YAML, no header. Top level: `checks` (yes, `[check]`) and `cases` (yes, `[case]`). Cases are
frozen inputs: a change invalidates their evidence.

Check:

| Field | Req | Type and rule |
| --- | --- | --- |
| `id` | yes | `check_id` |
| `family` | yes | `check_family` |
| `subjects` | yes | `subjects` |
| `components` | yes | `[component_id] unique` |
| `operations` | yes | `[semantic_id] unique` |
| `requirements` | — | `requirement_ids` |
| `criterion` | yes | `criterion_or_unknown` |
| `method` | yes | `text` |
| `facet` | yes | `facet` |
| `due` | yes | `stage` |
| `applicability` | yes | `{status APPLICABLE\|CONDITIONAL\|NOT_APPLICABLE, reason text}` required; `condition text` when CONDITIONAL; `reviewed_by text` when NOT_APPLICABLE |
| `safety` | yes | boolean |
| `waiver_policy` | yes | `waiver_policy`; `FORBIDDEN` when `safety` is true |
| `cases` | — | `[case_id]≥1 unique` |
| `producer_role` | yes | `producer_role` |

Case, every kind: `id case_id`, `kind`, `subjects subjects` required; `title text`, `requirements requirement_ids`
optional; `state text` and `criterion criterion_or_unknown` required except on MASS, where they are optional.

| `kind` | Fields (all required unless marked optional) |
| --- | --- |
| `MOTION` | `joints [semantic_id]≥1 unique`; `sampling sampling` (CONSTRAINED_PATH and SAMPLED state a `justification`); `steps [{joint semantic_id, step quantity}]≥1`; `pairs` `ALL` or `[[semantic_id, semantic_id]]≥1` (each pair exactly two); `play_extremes` boolean; optional `tool_state`, `service_state` `text` |
| `STACK` | `contributors [contributor]≥1`; `method WORST_CASE\|JUSTIFIED_RSS`; `rss` when `JUSTIFIED_RSS`; optional `interface interface_id`. Contributor `{id semantic_id, subject subject, sign 1\|-1, nominal quantity, tolerance quantity, state dimension_state, source source}`, optional `effects texts`. `rss {distribution text, independence text, centering text, sigma number > 0, confidence number in (0, 1], capability text, decision decision_id}`, all required |
| `LOAD` | `loads [load]≥1`; `boundary_conditions [boundary condition]≥1`; `material`; `failure_modes texts`; `method HAND\|FEA`; `validity text` |
| `FEA` | `solver {name text, version text, documentation text}` required, optional `capability profile_id` inside `solver`; `loads [load]≥1`; `boundary_conditions [boundary condition]≥1`; `material`; `mesh {element text, seed quantity, threads integer ≥ 1, refinement texts}`; `convergence {quantity semantic_id, tolerance quantity, levels integer ≥ 2}`; `units =mm-N-MPa`; `limits texts`; optional `contact texts` |
| `MASS` | `densities [{subject subject, density quantity_fact}]≥1`; `outputs [MASS\|CENTRE_OF_MASS\|INERTIA]≥1 unique`; optional `stability text` |

Boundary condition: `{id semantic_id, kind FIXED|PINNED|SYMMETRY|CONTACT|SPRING|OTHER, subject subject, description
text}`, all required. Material (LOAD, FEA): `{profile profile_id, properties [semantic_id]≥1 unique, basis_required
ALLOWABLE|SUPPLIER_TYPICAL|MEASURED}` required, `state text` optional.

## material-library

Wrapper: `profiles` (yes, `[root:material-profile]`). Each item validates against the `material-profile` schema
root. The starter assets hold one profile per file in this wrapper; copy the item into
`hw/manufacturing/materials.yaml`.

## capability-profile

What an actual machine, provider or operator can do, apart from design requirements. Parameters never state
achieved accuracy; only calibrations do. A plugin template carries `template: true` and is never a capability.

| Field | Req | Type and rule |
| --- | --- | --- |
| `id` | yes | `profile_id` |
| `revision` | yes | integer ≥ 1 |
| `title` | yes | `text` |
| `process` | yes | `route_kind` |
| `provider` | yes | `LOCAL\|OUTSOURCED` |
| `template` | — | `=true`, only on a plugin template |
| `guidance` | iff `template` | `texts` |
| `supplier`, `operator` | — | `text_fact` |
| `machine` | yes | `{identity text_fact}` required, `location text` optional |
| `envelope` | — | `{x, y, z}`, each `quantity_fact`, all required |
| `materials` | yes | `[approval]≥1` or `fact_open` |
| `parameters` | yes | map of `semantic_id` keys to `fact` values |
| `tools` | — | `[{id semantic_id, description text, parameters}]≥1`, all required; `parameters` as above |
| `workholding` | — | `text_fact` |
| `constraints` | — | `texts` |
| `calibrations` | yes | `[calibration]` |
| `source` | — | `source` |

Template (`template: true`): `tools` and `source` absent; `calibrations` empty; `materials` is `fact_unknown`; every
`parameters` value, `machine.identity`, and any `operator`, `supplier` and `envelope.x`/`y`/`z` are `fact_unknown`.

Approval: `{material profile_id, status APPROVED|FORBIDDEN, source source}` required; optional `thickness
quantity_fact`, `diameter quantity_fact` (the approved filament, wire, bar or tube diameter), `coating text_fact`,
`extraction text_fact`, `operations [CUT|SCORE|ENGRAVE|MACHINE|PRINT|FORM|JOIN|OTHER]≥1 unique`; `reason text` when
FORBIDDEN.

Calibration, one demonstrated capability: `{id semantic_id, characteristic semantic_id, scope, result, source source,
date date}` required; optional `valid_until date`, `decision decision_id`. `scope {material profile_id}` required with
optional `condition`, `feature`, `orientation`, `setup`, `finish` (`text`), `size`, `thickness` (`quantity`) and
`tool semantic_id`. `result {limits quantity, samples integer ≥ 1}` required with optional `distribution text` and
`uncertainty quantity`.

## capability-library

Wrapper: `profiles` (yes, `[capability-profile]`). Each item conforms to [capability-profile](#capability-profile).

## selection

Authored record YAML, no header. Hard requirement outcomes decide feasibility; preferences rank feasible candidates
only; the recommendation is NONE, with no route, exactly when no candidate is FEASIBLE
([cad-select Recommend](../../cad-select/references/selection.md#recommend)); a FINAL recommendation names its
approving decision.

| Field | Req | Type and rule |
| --- | --- | --- |
| `status` | yes | `NOT_ASSESSED\|ASSESSED\|CONFIRMED` |
| `assessment` | iff ASSESSED or CONFIRMED | `{inputs, input_sha256 sha256, method text, date date}` required; `decision decision_id` when CONFIRMED. `inputs [{path hw_path, sha256 sha256}]≥1 unique`, all required: every `hw/` file the assessment read, never `hw/manufacturing/selection.yaml` or a decision written from this assessment; `input_sha256` is the set digest over `inputs` |
| `candidates` | yes | `[candidate]`; empty when NOT_ASSESSED, ≥1 otherwise |
| `recommendation` | iff ASSESSED or CONFIRMED | recommendation object (below); `status` FINAL when CONFIRMED |

Candidate:

| Field | Req | Type and rule |
| --- | --- | --- |
| `id` | yes | `slug` |
| `title`, `architecture` | yes | `text` |
| `parts` | yes | `[{PN pn, sourcing sourcing, material_profile, route}]≥1` required, optional `title text` and `proposed` boolean (true when the candidate introduces the PN); `material_profile` is `profile_id` or `fact_open`; `route [step]≥1` |
| `hard_outcomes` | yes | `[hard outcome]≥1` |
| `feasibility` | yes | `FEASIBLE\|CONDITIONAL\|EXCLUDED` |
| `preference_scores` | — ; never when CONDITIONAL or EXCLUDED | `[{preference preference_id, score number, reason text}]≥1`, all required |
| `economics` | yes | `{quantity quantity_fact, cost money_fact, lead_time quantity_fact, maintenance text_fact}`, all required |
| `limits` | yes | `texts` |
| `reversal` | yes | `text` |
| `pros`, `cons` | — | `texts` |
| `tradeoffs` | — | `text` |

- Route step: `kind route_kind` required, optional `provider LOCAL|OUTSOURCED`; `capability` (`profile_id` or
  `fact_unknown`) iff `kind` is not PURCHASED; `purchase` (`part#purchase`) iff `kind` is PURCHASED.
- Hard outcome (also used by [variant](#variant)): `{requirement requirement_id, outcome PASS|FAIL|UNKNOWN|PLANNED,
  criterion criterion_or_unknown, method text}` required; PASS and FAIL need `source source` and a real `criterion`;
  UNKNOWN needs `reason text` and `question question_id`, the open question that settles it; PLANNED needs
  `reason text`, a real `criterion` and `due stage`, never GROUND or INTENT. `question` appears only on UNKNOWN and
  `due` only on PLANNED. What PLANNED means: [cad-select assess-selection](../../cad-select/references/selection.md#assess-selection).
- Feasibility: FEASIBLE has no FAIL or UNKNOWN hard outcome; CONDITIONAL has at least one UNKNOWN and no FAIL;
  EXCLUDED has at least one FAIL. A score or weighting never ranks a FAIL feasible.
- Recommendation: `{status NONE|CONDITIONAL|FINAL, governing_requirements requirement_ids, reasons texts, limits
  texts, reversal text, what_would_change texts}` required; `primary` (`slug`) iff not NONE; optional `prototype`,
  `end_use`, `fallback` (`slug`), never when NONE; `decision decision_id` when FINAL, never when NONE. `status` is
  NONE iff no candidate is FEASIBLE.

## variant

Authored by a concept unit and merged unchanged. A preview is never verification.

| Field | Req | Type and rule |
| --- | --- | --- |
| `id` | yes | `variant_id` |
| `title`, `architecture` | yes | `text` |
| `architecture_doc` | — | `hw_path` |
| `parts` | yes | `[{PN pn, title text, sourcing sourcing, material_profile, route}]≥1`, all required; `material_profile` is `profile_id` or `fact_open`; `route` is `[route_kind]≥1` or `fact_unknown` |
| `requirements` | yes | `requirement_ids` |
| `hard_outcomes` | yes | `[hard outcome]≥1`, as in [selection](#selection) without `question`; a PLANNED `due` is a stage after VARIANT |
| `feasibility` | yes | `FEASIBLE\|CONDITIONAL\|EXCLUDED`, the selection feasibility rule |
| `dependencies` | — | `texts` |
| `unknowns` | yes | `[fact_unknown]` |
| `limits` | yes | `texts` |
| `source` | yes | `{unit unit_id, role role, brief_sha256 sha256}`, all required, copied from the brief; never the delivery digest, because the delivery's `outputs` hash this file |
| `geometry` | yes | `{status =PRESENT, kernel kernel, files [file]≥1, units =mm}` or `{status =NOT_APPLICABLE, reason text}`; never a placeholder STEP |

Geometry file (also used by [version-manifest](#version-manifest)): `{path hw_path, sha256 sha256, role
DEFINITION|MANUFACTURE|VISUALIZATION|PREVIEW|PROFILE|PROXY, format STEP|STL|GLB|SVG|PNG|DXF}` required; optional `PN
pn`, `component component_id`, `state semantic_id`.

## version-manifest

Generated JSON, written after geometry build and readback (geometry NOT_APPLICABLE before CAD). It excludes itself,
check results, the stage verdict, later decisions and every drawing, BOM or verification output that embeds its
digest. `<vN>/` below is a path prefix matching `hw/versions/v[1-9][0-9]*/`.

| Field | Req | Type and rule |
| --- | --- | --- |
| `format` | yes | `=cad-agent-skills.version-manifest` |
| `schema_version` | yes | `=1` |
| `version` | yes | `version` |
| `stage` | yes | `stage` |
| `predecessor` | when `stage` is not GROUND | `{stage stage, version version, verdict file_hash}`, all required |
| `record` | yes | `[file_hash]≥1`, each path under `<vN>/record/` and never ending in `manifest.json` or `stage-verdict.json` |
| `cases` | yes | `[case_id] unique` |
| `build` | yes | `{status =PRESENT, source, locks}` with `source` and `locks` each `[file_hash]≥1` under `<vN>/source/`, no path ending in `manifest.json` or `stage-verdict.json`; or `{status =NOT_APPLICABLE, reason text}` |
| `source_artifacts` | yes | `[file_hash]`, each path under `<vN>/record/` |
| `runtime` | yes | `{status =PRESENT, environments, platform platform, commands [command]≥1}` or `{status =NOT_APPLICABLE, reason text}` |
| `geometry` | yes | `{status =PRESENT, units =mm, frame =RIGHT_HANDED_Z_UP, files, occurrences, export_tolerance, readback}` or `{status =NOT_APPLICABLE, reason text}` |
| `manufacturing` | yes | `[{PN pn, relationship SAME\|DIFFERENT, components [component_id] unique, files [file_hash]≥1}]` required, each file under `<vN>/manufacturing/` and never ending in `manifest.json` or `stage-verdict.json`; optional `state semantic_id`, `orientation quaternion` |

- `runtime.environments`: `[{kernel kernel, python text, packages [package]≥1, lock file_hash}]≥1`, all required,
  `lock` under `<vN>/source/`, at most one environment per kernel; never a top-level kernel, python or packages.
- `geometry.files`: `[geometry file]≥1` ([variant](#variant)), each under `<vN>/definition/`, `<vN>/manufacturing/` or
  `<vN>/previews/`.
- `geometry.occurrences`: `[{occurrence semantic_id, PN pn, rev part_rev, translation vector, rotation quaternion,
  step_name text}]≥1`, all required; `step_name` is the exported STEP product name, mapped back on readback.
- `geometry.export_tolerance`: `{linear_deflection_mm number > 0, angular_deflection_rad number in (0, π], glb
  =METRE_Y_UP}`, all required.
- `geometry.readback`: `{solids integer ≥ 0, occurrences integer ≥ 0, valid boolean, method text}`, all required.

## bundle-manifest

Generated JSON inventory of one result or output bundle; it excludes itself.

| Field | Req | Type and rule |
| --- | --- | --- |
| `format` | yes | `=cad-agent-skills.bundle-manifest` |
| `schema_version` | yes | `=1` |
| `kind` | yes | `CHECK\|DRAWING\|BOM\|VERIFICATION` |
| `binding` | yes | `binding` |
| `input` | yes | `[file_hash]≥1` |
| `recipe` | yes | `{kind =COMMAND, commands [command]≥1, packages [package], platform platform}`, all required; or `{kind =REVIEW, inspected [file_hash]≥1}`, which has no commands |
| `policy` | yes | `installed_policy`: path and plugin name and version, never a hash |
| `files` | yes | `[{path, sha256 sha256, role}]≥1` required, optional `bytes` integer ≥ 0; `path` is an `hw_path` (never scratch) never ending in `bundle-manifest.json`; role `RESULT\|SOURCE\|LOCK\|ARTIFACT\|LOG\|DOCUMENT\|DATA\|IMAGE`; a CHECK bundle lists at least one RESULT file |
| `complete` | yes | boolean |

## bom

Generated YAML: the typed EBOM of one version. `bom.csv` derives from this file only.

| Field | Req | Type and rule |
| --- | --- | --- |
| `format` | yes | `=cad-agent-skills.bom` |
| `schema_version` | yes | `=1` |
| `version` | yes | `version` |
| `version_manifest_sha256` | yes | `sha256` of the bound version manifest |
| `rows` | yes | `[root:bom-row]`; each item validates against the `bom-row` schema root; sorted by path then component |
| `totals` | yes | `{mass quantity_fact, part_count integer ≥ 0}` required, optional `price money_fact` |

## verification

Generated YAML: the derived verification index of one version. Check results stay the sole observation
authority; `verification.csv` derives from this file only.

| Field | Req | Type and rule |
| --- | --- | --- |
| `format` | yes | `=cad-agent-skills.verification` |
| `schema_version` | yes | `=1` |
| `version` | yes | `version` |
| `stage` | yes | `stage` |
| `version_manifest_sha256` | yes | `sha256` |
| `rows` | yes | `[verification-row]`, sorted by requirement, subject anchor, facet, then criterion `id`, else `property`, an UNKNOWN criterion last |

## verification-row

One derived requirement × subject × facet row. Outcomes are copied from check results, never invented; PLANNED marks
a facet due at a later stage.

| Field | Req | Type and rule |
| --- | --- | --- |
| `requirement` | yes | `requirement_id` |
| `subject` | yes | `subject` |
| `method` | yes | `verification_method` |
| `criterion` | yes | `criterion_or_unknown`; a real `criterion` when PASS |
| `facet` | yes | `facet` |
| `due` | yes | `stage` |
| `outcome` | yes | `PASS\|FAIL\|UNKNOWN\|NOT_APPLICABLE\|WAIVED\|PLANNED` |
| `obligations` | yes | `[obligation_id] unique`; never a `verification.index@…` obligation, and no row cites that check's result file; empty only with `outcome` UNKNOWN and `unknown_code` `INCOMPLETE` (a scope gap) |
| `results` | when PASS, FAIL, NOT_APPLICABLE or WAIVED; never when PLANNED | `[file_hash]≥1` |
| `waiver` | when WAIVED | `decision_id` |
| `reason`, `unknown_code` | when UNKNOWN | `text`, `unknown_code` |

## stage-policy

The installed stage policy (cad-flow `assets/stages.yaml`): ordered stages with predecessor, gate, owner, procedure
IDs, registered check families with applicability, due stage and reopen policy, and the fixed cost order of check
families.

| Field | Req | Type and rule |
| --- | --- | --- |
| `format` | yes | `=cad-agent-skills.stage-policy` |
| `schema_version` | yes | `=1` |
| `stages` | yes | `[stage entry]`, exactly 13 |
| `check_order` | yes | `[check_family]≥1 unique` |

Stage entry: `{id stage, order integer 1–13, predecessor (stage or null), gate HUMAN|NONE|REVIEW, owner skill,
procedure, checks, reopen}`, all required; `gate_subject text` when the gate is HUMAN or REVIEW. `procedure` is
`[string matching ^[a-z][a-z0-9_-]*$]≥1 unique`; `reopen` is `[stage] unique`. Stage IDs are the stage enum, with no
alias. Order and predecessor consistency is a semantic check.

Stage check: `{family check_family, applicability ALWAYS|CONDITIONAL, due stage}` required; `condition text` when
CONDITIONAL; optional `required_by stage`, `safety` boolean and `waiver_policy waiver_policy`.

## worker-brief

Generated YAML: the self-contained assignment of one worker.

| Field | Req | Type and rule |
| --- | --- | --- |
| `format` | yes | `=cad-agent-skills.worker-brief` |
| `schema_version` | yes | `=1` |
| `unit` | yes | `unit_id` (never `lead` or `stage-*`) |
| `role` | yes | `role` (the five roles) |
| `stage` | yes | `stage` |
| `version` | — | `version` |
| `task` | yes | `text` |
| `subjects` | yes | `subjects` |
| `seed_urls` | iff `role` is `cad-sourcing-researcher` | `[url]` unique; empty only when the host gives the role a search tool |
| `input_inventory` | yes | `{sha256 sha256, files}`: `sha256` is the set digest over `files`; `files [{path hw_path, sha256 sha256}]≥1` required, optional `bytes` integer ≥ 0; optional `snapshots [{path, sha256 sha256}]` unique, optional `bytes` integer ≥ 0, `path` `lead/attempts.txt` or `installed/<skill>/<path>` with `<skill>` one of the eight `cad-*` skills and `<path>` relative under the safe-path rules of `hw_path`, never absolute; a snapshot sha256 binds the copy as a check input, never an install audit |
| `resources` | yes | `{plugin plugin, skills, rubric, schemas, tools, templates, role_reference}`, all required, optional `layout absolute_path` and `policy absolute_path` (the installed `layout.json` and stages policy, for a unit that runs `record.semantic`): literal resolved absolute paths from the lead preflight; installed bytes are never hashed |
| `boundary` | yes | `{project_root absolute_path, scratch scratch_path, writable [scratch_path]≥1 unique, locked [hw_path] unique}` required, optional `read_only [project_path] unique`; a worker writes only under `.cad-work/` |
| `limits` | yes | `{wall_seconds integer ≥ 1, model_turns integer ≥ 1}`, both required |
| `expected` | yes | `{outputs, checks [check_id] unique, stop texts, report text}` required, optional `obligations [obligation_id] unique`, required when `role` is `cad-checker` or `cad-mech-dfm-reviewer` |
| `lead` | yes | `{session text}` required, optional `task_id text` |

- `resources.skills`: `[{name skill, path absolute_path}]≥1` required, optional `uri` matching
  `^skill://cad-[a-z]+(/[A-Za-z0-9._/-]+)?$`; no sha256.
- `resources.schemas`: `[{name, path absolute_path}]`, all required; `name` is `common` or one of the ten schema roots
  listed at the top. A prose root is never listed; its contract reaches the worker through the cad-record skill.
- `resources.role_reference`: `{name text, path absolute_path}` required, optional `uri` (same pattern).
- `resources.rubric`: `{name =rubric, path absolute_path}` required, `path` ending in `/cad-review/references/rubric.md`; optional
  `uri =skill://cad-review/references/rubric.md`.
- `resources.tools`: `[{name, argv, version text}]` unique, all required; `name` is `check-jsonschema` or `yaml-loader`; `argv` is
  `[text]≥1` whose first item is an `absolute_path`. Every role except `cad-mech-dfm-reviewer` lists `check-jsonschema`.
- `resources.templates`: `[{name text, path absolute_path}]≥1` unique, all required; `path` ends in
  `/cad-flow/assets/worker-delivery.yaml` or `/cad-record/assets/contract-fixtures/check-result/valid/<name>.json`.
- `expected.outputs`: each item is either `{target hw_path, kind text}` or, only when `role` is `cad-sourcing-researcher`,
  `{target_pattern, kind text, max integer 1–8}` with `target_pattern` `hw/manufacturing/sources/<sha256>/<name>` or
  `hw/parts/<PN>/sources/<sha256>/<name>` (literal placeholders); both forms are closed and all their fields required.
- `resources.plugin` is name and version from the cad-flow SKILL.md metadata; no manifest path, no hash.

## worker-delivery

Generated JSON: what one worker returns. A failed build or a FAIL or UNKNOWN check is never a ready PASS; the lead
copies accepted bytes verbatim and rechecks.

| Field | Req | Type and rule |
| --- | --- | --- |
| `format` | yes | `=cad-agent-skills.worker-delivery` |
| `schema_version` | yes | `=1` |
| `unit` | yes | `unit_id` |
| `role` | yes | `role` |
| `brief_sha256`, `input_inventory_sha256` | yes | `sha256` |
| `verdict` | yes | `PASS\|MINOR\|MAJOR` |
| `build` | yes | `{status SUCCEEDED\|FAILED\|NOT_APPLICABLE}`; `log file_hash` when status is not NOT_APPLICABLE; `reason text` when NOT_APPLICABLE |
| `outputs` | yes | `[{source absolute_path, target hw_path, sha256 sha256, kind text}]`, all required |
| `checks` | yes | `[{result_file file_hash, result result, family check_family}]`, all required |
| `logs` | yes | `[file_hash]` |
| `boundary` | yes | `{method text, before_sha256 sha256, after_sha256 sha256, violations [text]}`, all required |
| `proposals` | yes | `[{target hw_path, path scratch_path, sha256 sha256, change text, reason text}]`, all required |
| `unknowns` | yes | `[fact_unknown]` |
| `limits` | yes | `[text]` |
| `ready` | yes | boolean |

- `build.status` FAILED → `verdict` MAJOR and `ready` false.
- Any `checks[].result` FAIL or UNKNOWN → `verdict` is not PASS and `ready` is false.
- At least one `boundary.violations` entry → `verdict` MAJOR and `ready` false.
- `ready` true → `verdict` PASS.

## lead-claim

The one live lead of a project, written after an exclusive `mkdir .cad-work/lead/active`. Age or PID alone never
proves the old owner stopped; recovery names an explicit reference.

| Field | Req | Type and rule |
| --- | --- | --- |
| `format` | yes | `=cad-agent-skills.lead-claim` |
| `schema_version` | yes | `=1` |
| `root` | yes | `absolute_path` |
| `host`, `session`, `owner` | yes | `text` |
| `harness` | yes | `OMP\|CLAUDE\|CODEX\|OPENCODE\|KIRO` |
| `started` | yes | `timestamp` |
| `resumes` | yes | `[{session text, time timestamp}]`, both required |
| `state` | yes | `ACTIVE\|RELEASED` |
| `released` | iff RELEASED | `timestamp` |
| `recovery` | — | `{reference text, operator text, previous_session text, time timestamp}`, all required |

## layout

The installed path catalog (`<cad-record>/layout.json`): tokens, path templates, schema root, owner skill, writer,
mutability, first stage, condition and gate credit of every runtime path.

| Field | Req | Type and rule |
| --- | --- | --- |
| `format` | yes | `=cad-agent-skills.layout` |
| `schema_version` | yes | `=1` |
| `tokens` | yes | map of keys matching `^[A-Z][A-Z_]*$` to `{pattern text, example text, meaning text}`, all required |
| `entries` | yes | `[entry]≥1` |

Entry, all fields required:

| Field | Type and rule |
| --- | --- |
| `id` | string matching `^[a-z][a-z0-9_]*(\.[a-z][a-z0-9_]*)+$` |
| `path` | string matching `^(hw\|\.cad-work)/[A-Za-z0-9_./{}*<>-]+$` |
| `kind` | `RECORD\|GENERATED\|SCRATCH\|NOTES` |
| `schema` | `<root>.schema.json` for one of the ten schema roots listed at the top, or null; null for every prose root; a string needs `format` YAML or JSON |
| `format` | `YAML\|JSON\|CSV\|STEP\|STL\|GLB\|SVG\|PDF\|PNG\|DXF\|PY\|MD\|BYTES\|DIRECTORY` |
| `owner` | `skill` |
| `writer`, `meaning` | `text` |
| `mutability` | `CONTROLLED\|IMMUTABLE\|APPEND_ONLY\|DERIVED\|SCRATCH\|NOTES` |
| `stage` | `stage` or `ANY` |
| `condition` | `{kind ALWAYS\|CONDITIONAL}` required, `when text` when CONDITIONAL |
| `gate_credit` | boolean; false for SCRATCH and NOTES |
