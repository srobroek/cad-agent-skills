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

- `record.schema` of vN re-hashes each earlier snapshot file against the sha256 that vK's `manifest.json` `record` lists
  for its path: one item-table row with basis `RECORDED`, root and sha256, PASS when they are equal, FAIL when they differ
  or the manifest does not list the path. A snapshot whose attempt wrote no manifest is bound by no hash and gets no row.
- A RECORDED row proves hash equality only, never shape: its message names vK's `record.schema` and `record.semantic`
  results and their outcome for the file, or says vK has none. It never becomes an UNCHANGED row of a later baseline.
- A semantic item reads an earlier snapshot only where [Inputs](#inputs) names one, and never judges its shape.

## Semantic check

JSON Schema does not prove items 1–16 and 18; item 17 stands in for the schema of each prose root. The lead or checker
writes its own program under `.cad-work/<unit>/`, or starts from a retained one
([cad-verify results — Implementation kind](../../cad-verify/references/results.md#implementation-kind) step 2), with an
explicit uv script lock, runs it against the inventoried input, and publishes a `record.semantic` check result; an
independent checker reads that program before its result counts.

The program reads only what the brief inventories and names ([Inputs](#inputs)). An item over a collection the record does
not have yet (assembly occurrences and joints before LAYOUT, deliveries, versions, BOM, verification index, runtime) holds,
its item-table row saying `no instance`. An input the brief lacks makes the items that need it UNKNOWN `MISSING_INPUT` and
is reported as a brief defect.

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
   published `hw/versions/vN/obligations.yaml` its `obligations` hashes, and that file follows the stage policy row: each family
   listed, or `inapplicable` with the reason its condition is false; future facets are `PLANNED`. A result whose obligation covers
   continuous travel (a MOTION case, or the approach or insertion of `motion.access` or `assembly.operations`) with any sampled pose set
   has `scope.coverage_kind` SAMPLED (PARTIAL when a scope item is missing), whatever its sampling kind; FULL is valid only for a finite
   scope ([cad-verify results — Coverage](../../cad-verify/references/results.md#coverage)).
9. Chain and authority: predecessor verdict and gate DR exist and bind, and the predecessor verdict's `result` is PASS; every
   `binding.verdict_result` equals the `result` of the verdict it hashes; a RELEASE DR binds a PASS RELEASE verdict of its
   `version` whose `release.inventory` result inspected the bound `proposal`; vN is the last line of `input/lead/attempts.txt`;
   approvals are USER or AUTHORITY statements whose `statement.path` file hashes to `transcript_sha256`; PRODUCTION and
   SAFETY_CRITICAL scope names a competent authority.
10. Waivers in a stage verdict, for every verdict in the inventory (the lead applies the same rules to its own verdict before
    writing it: [cad-flow write-verdict](../../cad-flow/references/stages.md#write-verdict) step 3):
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
12. Immutability and layout: published DRs, verdicts, manifests and results match their recorded hashes; every inventoried
    `hw/` file matches a layout entry (the whole tree is `record.ownership`'s:
    [cad-verify families](../../cad-verify/references/families.md#record-intake-and-selection)); no path field of evidence
    names a `.cad-work/` file, a command's `stdout` and `stderr` included
    ([cad-verify results — Implementation kind](../../cad-verify/references/results.md#implementation-kind) step 1); a
    command's `argv` and `cwd` record where it ran and are no evidence path.
13. Derived files: BOM rows sorted by path then component; verification rows by requirement, subject anchor
    ([contract index anchor grammar](contract-index.md)), facet, then criterion `id`, else `property`, an UNKNOWN criterion
    last; each CSV equals its YAML authority.
14. Bootstrap: after GROUND, `product.yaml` `id` and `title` differ from those of the installed template copy
    `input/installed/cad-record/assets/project-template/hw/product.yaml`.
15. Version runtime: each `runtime.environments[].lock` equals one `build.locks` entry (path and sha256), no
    two environments share a lock, and every built PN's `model.kernel` has the environment of that kernel.
16. Self-referencing evidence: no verification row cites a `verification.index` result file, and no variant,
    integrity result or index carries the digest of a file that hashes it (references/record.md hash graph).
17. Prose roots: each inventoried instance of a [prose root](prose-contract.md) outside an earlier snapshot
    ([Earlier snapshots](#earlier-snapshots)) conforms to its table, one item-table row per file:
    - every required field is present and no unlisted field appears, at every level;
    - constants, enums, array minimums, uniqueness and every `when`, `iff` and `never` condition and listed rule hold;
    - each typed value validates in-process against its named `common.schema.json` or `part.schema.json` `$defs`
      entry, and each `root:<name>` item (material library `profiles[]`, BOM `rows[]`) against that schema root,
      resolving `$ref`s from `<skill-dir>/schemas/`;
    - the stage policy's `order` runs 1–13 in stage-enum order, GROUND's `predecessor` is null and every other
      stage's `predecessor` is the stage before it.
18. Program provenance: no `cad-checker` result's program (a `source_files` entry not ending `.lock`) has the sha256 of a
    producer program in the record: a version build source under `hw/versions/v*/source/`, a part model
    `hw/parts/<PN>/model.py` or a drawing program `hw/drawings/v*/<SUBJECT>/source/build.py`. A program whose sha256
    equals a check program an earlier version's result retains is a reused one: some such result of the same family is
    listed in a stage verdict ([cad-verify results — Implementation kind](../../cad-verify/references/results.md#implementation-kind)
    steps 2 and 3).

### Inputs

The brief ([cad-flow worker brief](../../cad-flow/references/worker-brief.md#compose)) inventories every file below and names
this skill's schemas, `layout.json` and the stage policy; the lead's own facts are copies under `input/`
(`input_inventory.snapshots`). Every item reads the vN version `hw/versions/vN/**` and every `hw/decisions/DR-*.yaml` with
the statement and proposal files it hashes; these items read more:

| Items | Also read |
|---|---|
| 1, 2, 12, 17 | every inventoried file; 1, 2 and 17 skip an earlier snapshot ([Earlier snapshots](#earlier-snapshots)) |
| 3 | the `requirements.yaml` and `manufacturing/selection.yaml` snapshots of the version an APPROVED INTENT `GATE` DR binds |
| 7, 8, 10, 11, 12, 16, 18 | each result an inventoried verdict lists, CARRIED_FORWARD ones included: its `result.json`, `bundle-manifest.json`, `source/` files, each `artifacts[]` file, the published `stdout` and `stderr` logs named in `commands[]`, and every file its `inspected` lists (REVIEW results have no commands or artifacts; [cad-verify results — Implementation kind](../../cad-verify/references/results.md#implementation-kind) step 1) |
| 8, 9 | every earlier `hw/versions/v*/manifest.json`, `obligations.yaml` and `stage-verdict.json` |
| 9 | `input/lead/attempts.txt`: one `v<M>` line per attempt number in use when the lead wrote the brief, ascending |
| 13, 16 | `hw/bom/vN/**` and `hw/verification/vN/**` |
| 14 | `input/installed/cad-record/assets/project-template/hw/product.yaml` |
| 18 | every producer program: `hw/versions/v*/source/**`, `hw/parts/<PN>/model.py`, `hw/drawings/v*/<SUBJECT>/source/build.py` |

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
  aggregate's `value` is the tally as text (`42 of 42 files pass`, `18 of 18 items hold`).
  - `record.schema`: one row per universe file of its [scope](#scope), one per earlier snapshot file
    ([Earlier snapshots](#earlier-snapshots)) and one for the stage policy: file, root, sha256 (none for the stage policy,
    which is never hashed), basis `VALIDATED`, `UNCHANGED` or `RECORDED`, outcome and messages. A VALIDATED file is PASS
    when it loads (YAML) and, with a schema root, its root command lists it in `checked_paths` (exit 0) or `successes`;
    FAIL when the loader rejects it or the report lists it in `errors` or `parse_errors`, with every message and JSON path;
    UNKNOWN `NOT_RUN` when its command did not run or the report does not name it, never PASS.
  - `record.semantic`: one row per item (item number, outcome, files, message), item 17 one row per prose-root file.
- Each non-PASS file row or item is one finding: `record.schema:product:<locus>` or
  `record.semantic:item-<n>:<locus>`. `<locus>` is the file path, or the offending ID for an item that spans files,
  lower-cased, with every character outside `a-z0-9_` as `-` and every leading character outside `a-z` dropped
  (`hw-product-yaml`, `req-003`, `cad-work-u1-brief-yaml`). A FAIL finding names the file, JSON
  path and message.
