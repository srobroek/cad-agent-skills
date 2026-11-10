# Validation

Owner: cad-record. Two layers: the published `check-jsonschema` CLI proves the shape of the ten schema roots; a
model-written semantic check program proves what shape cannot, and the shape of the 17 prose roots
(references/prose-contract.md). Both report as check results (`record.schema`, `record.semantic`).

## Schema validation

Run check-jsonschema 0.39.0, the pinned published CLI, as preflight.json `tools[]` `check-jsonschema` argv
([cad-flow preflight](../../cad-flow/references/preflight.md#probe-tools)) from the project root; never resolve it through
`uvx` or another install. `<skill-dir>` is the absolute cad-record directory from the brief or preflight, never a guess.
Run one command per schema root over every instance of that root in [scope](#scope), each path quoted, in sorted order:

```sh
"<check-jsonschema argv>" --schemafile "<skill-dir>/schemas/decision.schema.json" -o json --verbose "hw/decisions/DR-001.yaml" "hw/decisions/DR-002.yaml"
```

- Exit 0 means every instance matches; the JSON report on stdout is then `{"status": "ok", "errors": [], "checked_paths": […]}`,
  `checked_paths` naming every file. Any other exit has `"status": "fail"`: each matching file in `successes`, one `errors`
  entry per violation (`filename`, `path`, `message`, `has_sub_errors`; a violation with sub-errors adds `best_match`,
  `best_deep_match`, `num_sub_errors` and `sub_errors`) and one `parse_errors` entry per file that did not parse
  (`filename`, `message`). Record stdout and stderr.
- Pass the root schema file itself; never `file#fragment`. Sibling `$ref`s resolve from the schema's own
  directory, so the installed tree must stay intact.
- Never pass an unquoted glob.
- A file whose `layout.json` entry names a `schema` validates against that root:

| Instance | Root |
|---|---|
| `hw/product.yaml` | product |
| `hw/requirements.yaml` | requirements |
| `hw/assembly.yaml` | assembly |
| `hw/parts/<PN>/part.yaml` | part |
| `hw/decisions/DR-###.yaml` | decision |
| `hw/versions/vN/stage-verdict.json` | stage-verdict |
| `hw/versions/vN/release-proposal.json`, `hw/releases/<REV>/release-manifest.json` | release-manifest |
| `hw/checks/vN/<check>/<attempt>/result.json`, `hw/drawings/vN/<subject>/coverage.json` | check-result |
| `hw/releases/<REV>/acceptance/package-integrity.json` | check-result |

These records have `schema: null` in `layout.json` and a prose root instead: interfaces, check plan, materials,
capabilities, selection, variants, version manifests, bundle manifests, BOMs, verification indexes, the lead claim,
briefs and deliveries, plus the installed `<cad-flow>/assets/stages.yaml`. Semantic item 17 checks them; no schema
command runs on them, and the [YAML loader](#yaml-loader) loads each YAML file among them.

A snapshot under `hw/versions/vN/record/` validates against the root, or conforms to the prose root, of the live
file it copies.

### YAML loader

The one YAML loader of the lead and every worker is PyYAML 6.0.3 `yaml.safe_load` on the file's bytes. check-jsonschema
parses YAML with ruamel.yaml, which accepts text this loader rejects (an unquoted `?` inside a flow mapping), so a schema
PASS alone never shows that a YAML file loads.

- Command: preflight.json `tools[]` `yaml-loader` argv
  ([cad-flow preflight](../../cad-flow/references/preflight.md#probe-tools)), every file quoted, in one call:

  ```sh
  "<yaml-loader argv>" -c 'import sys, yaml; [yaml.safe_load(open(p, "rb")) for p in sys.argv[1:]]' "hw/product.yaml" "hw/requirements.yaml"
  ```

  Exit 0 means every file loads. A non-zero exit names the first file that fails (`in "<path>", line …`): that file
  is FAIL, and the call repeats over the files after it until each file has loaded or failed.
- A program that reads record YAML (the lead's `checks.py`, a worker's `check.py` or `build.py`) declares
  `pyyaml==6.0.3` in its PEP 723 block and parses with `yaml.safe_load`; never another loader or a hand parser.
- `record.schema` loads every YAML instance in its scope before its root commands, and the lead runs the loader as the
  scratch diagnostic of [cad-flow freeze-inputs](../../cad-flow/references/stages.md#freeze-inputs) step 1.
- Authored YAML that loads: [record — Namespace and conventions](record.md#namespace-and-conventions).

### Scope

`record.schema` validates the version snapshot and every file published since the last accepted result; a file
hash-verified as unchanged is not validated again.

- Universe: every file of the vN snapshot `hw/versions/vN/record/`, and every other `hw/` file outside `hw/versions/v*/record/`
  whose `layout.json` entry names a schema or that is a YAML instance of a [prose root](prose-contract.md). A CONTROLLED live
  file (layout `mutability`) is validated as its snapshot copy, never in place. A prose-root YAML file is only loaded; its
  row's root is the prose root's name. An earlier attempt's snapshot is only re-hashed ([Earlier snapshots](#earlier-snapshots)).
- Baseline: the `record.schema` result of the newest `hw/versions/v*/stage-verdict.json` whose `policy.plugin` equals
  the installed plugin name and version, with its item table ([Reporting](#reporting)); both re-hash to the sha256s that
  verdict's `result_file` and the result's bundle manifest record. There is none at the first attempt, after a plugin
  change or when a re-hash differs.
- UNCHANGED: a universe file whose baseline row has the same path, root and sha256 and outcome PASS, and whose bytes
  still hash to that sha256. Its row is copied with basis `UNCHANGED`. Every other universe file, the vN snapshot
  among them, is validated: the YAML loader, then its root command when it has a schema root.
- The installed stage policy `<cad-flow>/assets/stages.yaml` is never hashed, so it is loaded in every attempt.
- The result's `input_files` list the baseline's `result.json` and item table, every universe file and every earlier
  snapshot file.
- Briefs, deliveries and the lead claim are validated where cad-flow writes or accepts them. A result published after
  `record.schema` ran is validated before publication
  ([cad-flow run-checks](../../cad-flow/references/stages.md#run-checks) step 3) and enters the next attempt's scope.

### Earlier snapshots

A file under `hw/versions/vK/record/`, where vK is not the attempt vN under check, is attempt vK's record. Only vK's own
checks judge its shape: its `record.schema` validates or loads it as vK's version snapshot and its `record.semantic` judges
its prose roots (item 17); those results' rows and findings and vK's verdict are the evidence of any defect. A later attempt
never validates it again and never rewrites or repairs it ([record — Mutability](record.md#mutability)): a defect is fixed
in the live file, which the next attempt snapshots and validates.

- `record.schema` of vN checks each earlier snapshot file by the [seal check](#seal-check): one item-table row with basis
  `RECORDED`, root and sha256, PASS when the file hashes to the sha256 that vK's `manifest.json` `record` lists for its path,
  FAIL when they differ. A snapshot whose attempt wrote no manifest is bound by no hash and gets no row.
- A path under vK's snapshot that vK's manifest does not list is no record file of vK. Its row has basis `REPORTED` and its
  sha256, and is PASS only when vK's own verdict carries a `HASH_MISMATCH` blocker whose `text` names that exact path; the
  row's message names that blocker. Any other unlisted path is FAIL: a change to published history that vK never reported.
  vK's own check fails the attempt that made the file, so a later attempt does not inherit that failure.
- A RECORDED row proves hash equality only, never shape: its message names vK's `record.schema` and `record.semantic`
  results and their outcome for the file, or says vK has none. Neither a RECORDED nor a REPORTED row is record evidence or
  becomes an UNCHANGED row of a later baseline.
- A semantic item reads an earlier snapshot only where [Inputs](#inputs) names one, and never judges its shape
  ([Attempt scope](#attempt-scope)).

## Semantic check

JSON Schema does not prove items 1–16, 18 and 19; item 17 stands in for the schema of each prose root. The lead or checker
writes its own program under `.cad-work/<unit>/`, or starts from a retained one
([cad-verify results — Implementation kind](../../cad-verify/references/results.md#implementation-kind) step 2), with an
explicit uv script lock, runs it against the inventoried input, and publishes a `record.semantic` check result; an
independent checker reads that program before its result counts.

The program reads only what the brief inventories and names ([Inputs](#inputs)) and judges the attempt under check
([Attempt scope](#attempt-scope)). An item over a collection the record does not have yet (assembly occurrences and joints
before LAYOUT, deliveries, versions, BOM, verification index, runtime) holds, its item-table row saying `no instance`. An
input the brief lacks makes the items that need it UNKNOWN `MISSING_INPUT` and is reported as a brief defect.

1. Finite numbers: reject NaN and ±infinity anywhere, including YAML `.nan`, `.inf` and `-.inf`, before any
   other use; serialize JSON with non-finite numbers refused.
2. Uniqueness: every ID inside its collection and across files (PN, REQ, IF, DR, Q, ASM, PRF, case, check,
   variant, occurrence, joint, component, operation).
3. References: every cited ID, profile, case, check, obligation and path resolves to an existing entry. In `hw/manufacturing/selection.yaml`
   each recommendation route field (`primary`, `prototype`, `end_use`, `fallback`) names a candidate of that file whose `feasibility` is
   FEASIBLE; a FINAL recommendation's `decision` is an APPROVED INTENT `GATE` DR whose `choice` is the `primary` and whose bound version's
   snapshot of this file has the same `candidates` and `assessment.input_sha256`. Each UNKNOWN hard outcome's `question` is OPEN
   in the `hw/requirements.yaml` of the same snapshot. A MOTION case's `joints` and `steps[].joint` and an interface member's
   `occurrence` resolve in `hw/assembly.yaml` from LAYOUT on, where cad-design writes joints and placements; before LAYOUT
   they have no instance.
4. Assembly tree: one root, every parent exists, no cycle; joints connect existing occurrences.
5. Order and units: `lower <= upper`, BETWEEN bounds ordered, joint limits ordered; a value, its criterion and its bounds share one
   dimension of [Unit dimensions](#unit-dimensions); quaternions have unit norm within 1e-9 and axes are non-zero.
6. Hashes: every recorded sha256 equals the bytes on disk; a retained source's directory name equals its
   sha256; inventories exclude themselves and every downstream file (references/record.md hash graph); a check bundle
   manifest lists exactly the files [record — Bundle manifest](record.md#bundle-manifest) names.
7. Freshness: a VERSION result's inputs, cases and check plan equal the version snapshot; a CARRIED_FORWARD result keeps its
   original binding and instead equals the current bytes input by input, and its outcomes are all PASS or NOT_APPLICABLE
   ([cad-flow change impact — Carry-forward](../../cad-flow/references/change-impact.md#carry-forward)); calibrations and waivers
   are unexpired at the verdict time; a changed input makes the old evidence stale (UNKNOWN, `STALE`).
8. Coverage: a result's outcomes cover exactly its `expected_obligations`; a stage verdict's `expected` and `future` equal the
   published `hw/versions/vN/obligations.yaml` its `obligations` hashes, and that file follows its stage row: the installed
   `stages` row for a FULL attempt, its captured `policy.row` for a LITE attempt (item 19): each family
   listed, or `inapplicable` with the reason its condition is false; future facets are `PLANNED`. A result whose obligation covers
   continuous travel (a MOTION case, or the approach or insertion of `motion.access` or `assembly.operations`) with any sampled pose set
   has `scope.coverage_kind` SAMPLED (PARTIAL when a scope item is missing), whatever its sampling kind; FULL is valid only for a finite
   scope ([cad-verify results — Coverage](../../cad-verify/references/results.md#coverage)).
9. Chain and authority: the predecessor verdict exists and binds ([Seal check](#seal-check)) and its `result` is PASS, and
   its gate DR exists and binds when the predecessor's stage row (captured in its `obligations.yaml` for a LITE attempt) has
   gate HUMAN; every `binding.verdict_result` equals the `result` of the verdict it hashes; a RELEASE DR binds a PASS RELEASE
   verdict of its `version` whose `release.inventory` result inspected the bound `proposal`; vN is the last line of
   `input/lead/attempts.txt`; approvals are USER or AUTHORITY statements whose `statement.path` file hashes to
   `transcript_sha256`; PRODUCTION and SAFETY_CRITICAL scope names a competent authority.
10. Waivers in a stage verdict, the verdict the lead composes ([Attempt scope](#attempt-scope)):
    - Existence: each `WAIVED` result's `WV-###` is the `waiver.id` of an APPROVED WAIVER DR that no later
      DR rejects or supersedes, and the verdict's `waivers` list equals the set of waivers its results cite.
    - Scope coverage: the waiver's obligation and subject cover every expected obligation of that check that
      is not PASS; the check's own `result.json` result is FAIL or UNKNOWN and equals `retained_outcome`; no
      covered obligation is safety or regulatory, and neither its requirement, its check plan entry nor the stage policy's
      `checks[].waiver_policy` for that family and stage is `FORBIDDEN`.
    - Expiry: each condition the waiver states holds at the verdict: `expires` is after the verdict `time`, and every
      `until_change` file has its recorded sha256 in the verdict's version snapshot.
    - Every other result's `result` equals the `result` in its `result.json`.
11. Digests: every `input_inventory` and `boundary` digest recomputes exactly by
    references/record.md#inventory-digest; `assessment.input_sha256` is the set digest over `assessment.inputs`, and an input
    whose current bytes differ makes the selection `STALE`; a REVIEW implementation's `inspected` hashes match the bytes read,
    and `inspected` never lists the review's own `result.json` or `bundle-manifest.json`.
12. Immutability and layout: published DRs, verdicts, manifests and results match their recorded hashes, an earlier
    attempt's by the [seal check](#seal-check) and by nothing else ([Attempt scope](#attempt-scope)); every `hw/` file of the attempt
    matches a layout entry (the whole tree is `record.ownership`'s:
    [cad-verify families](../../cad-verify/references/families.md#record-intake-and-selection)); every
    [path field](#path-fields) of the attempt's results is a published `hw/` path.
13. Derived files: BOM rows sorted by path then component; verification rows by requirement, subject anchor
    ([contract index anchor grammar](contract-index.md)), facet, then criterion `id`, else `property`, an UNKNOWN criterion
    last; each CSV equals its YAML authority.
14. Bootstrap: after GROUND, `product.yaml` `id` and `title` differ from those of the installed template its build mode
    selects ([record — record-bootstrap](record.md#record-bootstrap)), the brief snapshot
    `installed/cad-record/assets/project-template/hw/product.yaml` or
    `installed/cad-record/assets/project-template-lite/hw/product.yaml`. The item's row names that template and the
    brief's sha256 for it in its message only, never in `files` ([Path fields](#path-fields)).
15. Version runtime: each `runtime.environments[].lock` equals one `build.locks` entry (path and sha256), no
    two environments share a lock, and every built PN's `model.kernel` has the environment of that kernel.
16. Self-referencing evidence: no verification row cites a `verification.index` result file, and no variant,
    integrity result or index carries the digest of a file that hashes it (references/record.md hash graph).
17. Prose roots: each inventoried instance of a [prose root](prose-contract.md), except an earlier attempt's file
    ([Attempt scope](#attempt-scope)), conforms to its table, one item-table row per file:
    - every required field is present and no unlisted field appears, at every level;
    - constants, enums, array minimums, uniqueness and every `when`, `iff` and `never` condition and listed rule hold;
    - each typed value validates in-process against its named `common.schema.json` or `part.schema.json` `$defs`
      entry, and each `root:<name>` item (material library `profiles[]`, BOM `rows[]`) against that schema root,
      resolving `$ref`s from `<skill-dir>/schemas/`;
    - the stage policy's `order` runs 1–13 in stage-enum order, GROUND's `predecessor` is null and every other
      stage's `predecessor` is the stage before it; `lite_stages` holds its three fixed entries.
18. Program provenance: no `cad-checker` result's program (a `source_files` entry not ending `.lock`) has the sha256 of a
    producer program in the record: a version build source under `hw/versions/v*/source/`, a part model
    `hw/parts/<PN>/model.py` or a drawing program `hw/drawings/v*/<SUBJECT>/source/build.py`. A program whose sha256
    equals a check program an earlier version's result retains is a reused one: some such result of the same family is
    listed in a stage verdict ([cad-verify results — Implementation kind](../../cad-verify/references/results.md#implementation-kind)
    steps 2 and 3).
19. Build mode ([record — Build mode](record.md#build-mode)):
    - Mode: vN's snapshot `hw/product.yaml` has a KNOWN `build_mode`. An earlier snapshot without one is a legacy FULL
      record; it is read as FULL and never judged by this item. The mode differs from the predecessor's only through an
      APPROVED CHANGE DR that reopens GROUND.
    - Discriminators: every `part.yaml` of vN's snapshot has `build_mode: LITE` exactly when the product mode is LITE, and
      so does a release proposal or manifest of vN. A LITE manifest's `frozen` is the PASS LAYOUT verdict of the attempt's
      chain and its `scope.exclusions` holds the cad-flow disclosure line; a LITE PROPOSAL plans no RELEASE result,
      readiness verdict, approval or integrity file.
    - Profile: a LITE attempt's `obligations.yaml` has `build_mode: LITE` and `method_profile`, `policy.row`,
      `policy.check_order` and `policy.chain` equal to the installed `lite_stages` method profile, the `lite_stages` row of
      its stage, the installed `check_order` filtered to that row's families in installed order, and the `lite_stages`
      IDs in order; `policy.producers` follows [cad-flow run-checks](../../cad-flow/references/stages.md#run-checks) step 1.
      A FULL attempt's file has none of these keys. An earlier attempt's verdict is read against the profile its own
      `obligations.yaml` captured, never against today's rows.
    - Lite chain: a LITE attempt's predecessor is the PASS verdict of the `lite_stages` stage before its own (GROUND,
      LAYOUT, RELEASE), with a GATE DR only after GROUND (item 9).
    - Promotion: a FULL attempt after LITE attempts has an APPROVED LITE→FULL CHANGE DR that reopens GROUND and binds the
      lite baseline, `FROZEN` only to the PASS LAYOUT a sealed LITE FINAL froze; no live `part.yaml` keeps
      `build_mode: LITE`; a lite result enters a FULL verdict only by carry-forward under item 7; and no release REV letter
      repeats.

### Attempt scope

The semantic check of vN judges the attempt under check. An earlier attempt's files are history that no later attempt judges
or repairs ([record — Mutability](record.md#mutability)): a sealed attempt gets the [seal check](#seal-check) only, and an
unsealed one (failed, superseded or not yet reached by an approval) is only re-hashed. A sealed attempt was cleared once, at
its write-verdict and its gate, so a rule applied to it later would fail every later attempt on bytes nobody may change.

- Earlier files: each inventoried file of an earlier attempt vK, a path under `hw/versions/vK/`, `hw/checks/vK/`,
  `hw/bom/vK/`, `hw/verification/vK/` or `hw/drawings/vK/`: its record snapshot, manifest, obligations, verdict and release
  proposal, and each result an earlier verdict lists, CARRIED_FORWARD ones included. A published release
  `hw/releases/<REV>/**` is an earlier attempt's file too.
- The attempt's files: every other inventoried file. These are vN's own files, among them each inventoried result bound
  `VERSION` vN with its bundle, the live decision record (every `hw/decisions/DR-*.yaml` with the statement and proposal
  files it hashes), the live producer programs and the brief's snapshots.
- Each item judges the attempt's files. Item 12 checks each earlier file by the seal check, and no other item judges one.
  vK's own checks are the evidence of a defect in its files: its `record.schema` and `record.semantic` for its version
  ([Earlier snapshots](#earlier-snapshots)),
  [cad-flow worker-delivery Accept](../../cad-flow/references/worker-delivery.md#accept) for a delivered result,
  [cad-flow write-verdict](../../cad-flow/references/stages.md#write-verdict) for its verdict and
  [cad-release publish-release](../../cad-release/references/release.md#publish-release) for a release.
- Cross-version items read earlier files for the relation they state, never for their content: item 3 the snapshots an
  APPROVED INTENT `GATE` DR binds, item 9 the chain, item 18 the sha256 of earlier producer and retained check programs and
  item 19 the build-mode history.
- Clearance: vN's verdict is written after vN's `record.semantic` ran. Before the owner's gate, the lead applies the verdict
  items to the verdict it composes, to the results it lists and to each CARRIED_FORWARD result among them: items 2, 7, 8,
  10, 16 and 18, and item 12's path fields ([cad-flow write-verdict](../../cad-flow/references/stages.md#write-verdict)
  step 3). That check and the owner's review of the packet are the attempt's clearance; the approved decision then seals it.

### Seal check

An attempt is SEALED once an APPROVED decision binds its verdict: a `GATE` DR's `STAGE` binding, or a `RELEASE` DR's
`RELEASE_INVENTORY` binding with its proposal ([cad-flow gates](../../cad-flow/references/gates.md#record-a-gate-decision)).
The seal reaches every file the hash graph binds from that decision ([record — Acyclic hash graph](record.md#acyclic-hash-graph)):

- the DR's statement and proposal files and the verdict its binding names, with the release proposal for RELEASE;
- every `{path, sha256}` a reached verdict or version manifest records: the manifest, obligations, each result file and
  bundle manifest, the predecessor verdict and its approval, a superseded verdict, and the manifest's record, source and
  output files;
- every entry of a reached bundle manifest's `files`, never its `input`, which cites inputs that may change later.

The seal check walks that graph from each APPROVED `GATE` and `RELEASE` DR in `hw/decisions/`, over the files its check's
inputs hold: each reached file exists and hashes to the sha256 its binder records. A missing file or a different sha256 is
FAIL, its finding naming the file and its binder. A later DR that reopens or supersedes the stage never unseals what the
approval bound. An unsealed earlier file is re-hashed the same way against the sha256 that its own attempt's verdict,
manifest or bundle manifest records. Items 9 and 12 and the `record.schema` rows of [Earlier snapshots](#earlier-snapshots)
use this one check.

### Path fields

A path field is a published `hw/` path; an input snapshot or installed template is named only in a message and bound by the
brief's hashes.

- Path field: every string, at any depth, of a result, its bundle manifest and each JSON artifact the result cites
  (item-table rows included) that names a file: a string under a `path`, `file`, `files`, `source`, `target`, `stdout` or
  `stderr` key, and every other whitespace-free string that starts with `/`, `hw/`, `.cad-work/`, `input/`, `installed/` or
  `lead/`. A JSON path (`$…`) or a URL names no file.
- A file read through a copy is named by its origin, the `input_inventory.files` path (`hw/…`), never by its
  `.cad-work/<unit>/input/` copy. A `delivery.integrity` row names its unit and `hw/` target, never the unit's scratch
  source. A command's `stdout` and `stderr` name the logs beside the result
  ([cad-verify results — Implementation kind](../../cad-verify/references/results.md#implementation-kind) step 1).
- Not path fields: a command's `argv` and `cwd`, wherever recorded (`commands[]`, `freecad-run.json`), which say where it
  ran; the executable and home paths a run reports about itself (`sys.executable` and `FreeCAD.getHomePath()` in
  `freecad-readback.json`); and the installed stage policy path, in a bundle manifest's `policy.path` and in the
  stage-policy row of the `record.schema` item table.

### Inputs

The brief ([cad-flow worker brief](../../cad-flow/references/worker-brief.md#compose)) inventories every file below and names
this skill's schemas, `layout.json` and the stage policy; the lead's own facts are copies under `input/`
(`input_inventory.snapshots`). Every item reads the attempt's files ([Attempt scope](#attempt-scope)); these items read more:

| Items | Also read |
|---|---|
| 12, 19 | every inventoried file; item 12 reads an earlier file only for the [seal check](#seal-check) |
| 3 | the `requirements.yaml` and `manufacturing/selection.yaml` snapshots of the version an APPROVED INTENT `GATE` DR binds |
| 9, 19 | every earlier `hw/versions/v*/manifest.json`, `obligations.yaml` and `stage-verdict.json` |
| 9 | `input/lead/attempts.txt`: one `v<M>` line per attempt number in use when the lead wrote the brief, ascending |
| 12 | for the seal check, each result an inventoried verdict lists, CARRIED_FORWARD ones included: its `result.json`, `bundle-manifest.json`, `source/` files, each `artifacts[]` file, the published `stdout` and `stderr` logs named in `commands[]`, and every file its `inspected` lists (REVIEW results have no commands or artifacts; [cad-verify results — Implementation kind](../../cad-verify/references/results.md#implementation-kind) step 1) |
| 13, 16 | `hw/bom/vN/**` and `hw/verification/vN/**` |
| 14 | the brief snapshot `installed/cad-record/assets/project-template/hw/product.yaml` or `installed/cad-record/assets/project-template-lite/hw/product.yaml`, the template the build mode selects, read from its copy under `input/`; the row names it in its message only |
| 18 | every producer program: `hw/versions/v*/source/**`, `hw/parts/<PN>/model.py`, `hw/drawings/v*/<SUBJECT>/source/build.py` |
| 19 | every `hw/versions/v*/record/product.yaml` snapshot and `hw/releases/*/release-manifest.json` |

## Unit dimensions

Item 5 compares units within one row; a conversion never crosses rows.

| Dimension | Units |
|---|---|
| length | `mm`, `um`, `m` |
| area | `mm2` |
| volume | `mm3` |
| angle | `deg`, `rad` |
| force | `N`, `kN` |
| moment | `N_mm`, `N_m` |
| stiffness | `N_per_mm` |
| pressure and stress | `Pa`, `kPa`, `MPa`, `GPa`, `bar` |
| mass | `g`, `kg` |
| density | `kg_m3`, `g_cm3` |
| temperature | `deg_C`, `K` |
| time | `s`, `min`, `h`, `d` |
| count | `cycles`, `count` |
| frequency | `Hz` |
| angular speed | `deg_s`, `rpm` |
| linear speed | `mm_s`, `m_s` |
| ratio | `percent`, `dimensionless` |
| energy | `J` |
| energy per length | `J_m` |
| energy per area | `kJ_m2` |
| melt mass-flow rate | `g_10min` |
| melt volume-flow rate | `cm3_10min` |
| power | `W`, `kW` |
| thermal conductivity | `W_m_K` |
| thermal expansion | `um_m_K`, `per_K` |
| voltage | `V` |
| current | `A` |

## Reporting

`record.schema@product` and `record.semantic@product` each get one aggregate outcome, the only outcome of their
check result ([cad-verify results — Outcomes](../../cad-verify/references/results.md#outcomes)):

| Aggregate | When |
|---|---|
| `PASS` | every file row is PASS; every semantic item holds |
| `FAIL` | any file row is FAIL or any command exits non-zero; any item is violated |
| `UNKNOWN` | otherwise, when a file row or an item could not be assessed: the `unknown_code` all those share, else `INCOMPLETE` |

- Item table: `artifacts/items.json` in the check bundle, role `REPORT`. It is the aggregate's `evidence`; the
  aggregate's `value` is the tally as text (`42 of 42 files pass`, `19 of 19 items hold`).
  - `record.schema`: one row per universe file of its [scope](#scope), one per earlier snapshot file
    ([Earlier snapshots](#earlier-snapshots)) and one for the stage policy: file, root, sha256 (none for the stage policy,
    which is never hashed), basis `VALIDATED`, `UNCHANGED`, `RECORDED` or `REPORTED`, outcome and messages. A VALIDATED
    file is PASS when it loads (YAML) and, with a schema root, its root command lists it in `checked_paths` (exit 0) or
    `successes`; FAIL when the loader rejects it or the report lists it in `errors` or `parse_errors`, with every message
    and JSON path; UNKNOWN `NOT_RUN` when its command did not run or the report does not name it, never PASS.
  - `record.semantic`: one row per item (item number, outcome, files, message), item 17 one row per prose-root file.
- Each non-PASS file row or item is one finding: `record.schema:product:<locus>` or
  `record.semantic:item-<n>:<locus>`. `<locus>` is the file path, or the offending ID for an item that spans files,
  lower-cased, with every character outside `a-z0-9_` as `-` and every leading character outside `a-z` dropped
  (`hw-product-yaml`, `req-003`, `cad-work-u1-brief-yaml`). A FAIL finding names the file, JSON
  path and message.
