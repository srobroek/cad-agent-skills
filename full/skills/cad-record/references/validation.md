# Validation

Owner: cad-record. Two layers: the published `check-jsonschema` CLI proves shape; a model-written semantic
check program proves what shape cannot. Both report as check results (`record.schema`, `record.semantic`).

## Schema validation

Use the pinned published CLI, one schema and one instance per command, from the project root.
`<skill-dir>` is the absolute cad-record directory from the brief or preflight, never a guess.

```sh
uvx --from check-jsonschema==0.39.0 check-jsonschema --schemafile "<skill-dir>/schemas/product.schema.json" "hw/product.yaml"
```

- Exit 0 means the instance matches; any other exit is a FAIL for that file; record stdout and stderr.
- Pass the root schema file itself; never `file#fragment`. Sibling `$ref`s resolve from the schema's own
  directory, so the installed tree must stay intact.
- Quote every path. For a family (parts, decisions, results), run one command per literal path in sorted
  order; never pass an unquoted glob.
- Every file whose `layout.json` entry names a `schema` is validated against that root:

| Instance | Root |
|---|---|
| `hw/product.yaml` | product |
| `hw/requirements.yaml` | requirements |
| `hw/interfaces.yaml` | interfaces |
| `hw/assembly.yaml` | assembly |
| `hw/check-plan.yaml` | check-plan |
| `hw/manufacturing/materials.yaml` | material-library |
| `hw/manufacturing/capabilities.yaml` | capability-library |
| `hw/manufacturing/selection.yaml` | selection |
| `hw/parts/<PN>/part.yaml` | part |
| `hw/concepts/<variant>/variant.yaml` | variant |
| `hw/decisions/DR-###.yaml` | decision |
| `hw/versions/vN/manifest.json` | version-manifest |
| `hw/versions/vN/stage-verdict.json` | stage-verdict |
| `hw/versions/vN/release-proposal.json`, `hw/releases/<REV>/release-manifest.json` | release-manifest |
| `hw/checks/vN/<check>/<attempt>/result.json`, `hw/drawings/vN/<subject>/coverage.json` | check-result |
| `hw/releases/<REV>/acceptance/package-integrity.json` | check-result |
| every `bundle-manifest.json` | bundle-manifest |
| `hw/bom/vN/bom.yaml`, `hw/releases/<REV>/bom.yaml` | bom |
| `hw/verification/vN/verification.yaml`, `hw/releases/<REV>/verification.yaml` | verification |
| `.cad-work/lead/active/claim.yaml` | lead-claim |
| `.cad-work/<unit>/brief.yaml` | worker-brief |
| `.cad-work/<unit>/delivery.json` | worker-delivery |
| `<cad-flow>/assets/stages.yaml` | stage-policy |

A snapshot under `hw/versions/vN/record/` validates against the root of the live file it copies.

## Semantic check

JSON Schema does not prove the items below. The lead or checker writes its own program under
`.cad-work/<unit>/` with an explicit uv script lock, runs it against the inventoried input, and publishes a
`record.semantic` check result; an independent checker reads that program before its result counts.

The program reads the inventoried `hw/` files, including the statement files the DRs hash, this skill's schemas, `layout.json`
and the stage policy, all named in the brief ([cad-flow worker brief](../../cad-flow/references/worker-brief.md)).
An item over a collection the record does not have yet (assembly occurrences before LAYOUT, deliveries, versions,
BOM, verification index, runtime) holds, its item-table row saying `no instance`. An input the brief lacks makes the
items that need it UNKNOWN `MISSING_INPUT` and is reported as a brief defect.

1. Finite numbers: reject NaN and ±infinity anywhere, including YAML `.nan`, `.inf` and `-.inf`, before any
   other use; serialize JSON with non-finite numbers refused.
2. Uniqueness: every ID inside its collection and across files (PN, REQ, IF, DR, Q, ASM, PRF, case, check,
   variant, occurrence, joint, component, operation).
3. References: every cited ID, profile, case, check, obligation and path resolves to an existing entry. In `hw/manufacturing/selection.yaml`
   each recommendation route field (`primary`, `prototype`, `end_use`, `fallback`) names a candidate of that file whose `feasibility` is
   FEASIBLE; a FINAL recommendation's `decision` is an APPROVED INTENT `GATE` DR whose `choice` is the `primary` and whose bound version's
   snapshot of this file has the same `candidates` and `assessment.input_sha256`.
4. Assembly tree: one root, every parent exists, no cycle; joints connect existing occurrences.
5. Order and units: `lower <= upper`, BETWEEN bounds ordered, joint limits ordered; a value, its criterion and its bounds share one
   dimension of [Unit dimensions](#unit-dimensions); quaternions have unit norm within 1e-9 and axes are non-zero.
6. Hashes: every recorded sha256 equals the bytes on disk; a retained source's directory name equals its
   sha256; inventories exclude themselves and every downstream file (references/record.md hash graph).
7. Freshness: a VERSION result's inputs, cases and check plan equal the version snapshot; a CARRIED_FORWARD result keeps its
   original binding and instead equals the current bytes input by input
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
   `version` whose `release.inventory` result inspected the bound `proposal`; vN is unused and above every attempt; approvals
   are USER or AUTHORITY statements whose `statement.path` file hashes to `transcript_sha256`; PRODUCTION and
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
12. Immutability and layout: published DRs, verdicts, manifests and results match their recorded hashes;
    every `hw/` file matches a layout entry; no evidence cites `.cad-work/`.
13. Derived files: BOM rows sorted by path then component; verification rows by requirement, subject anchor
    ([contract index anchor grammar](contract-index.md)), facet, then criterion `id`, else `property`, an UNKNOWN criterion
    last; each CSV equals its YAML authority.
14. Bootstrap: after GROUND, `product.yaml` `id` and `title` are no longer the template values.
15. Version runtime: each `runtime.environments[].lock` equals one `build.locks` entry (path and sha256), no
    two environments share a lock, and every built PN's `model.kernel` has the environment of that kernel.
16. Self-referencing evidence: no verification row cites a `verification.index` result file, and no variant,
    integrity result or index carries the digest of a file that hashes it (references/record.md hash graph).

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
| `PASS` | every schema command exits 0; every semantic item holds |
| `FAIL` | any command exits non-zero, or any item is violated |
| `UNKNOWN` | otherwise, when a command did not run or an item could not be assessed: the `unknown_code` all those share, else `INCOMPLETE` |

- Item table: `artifacts/items.json` in the check bundle, role `REPORT`; one row per command (file, root, exit
  code, stderr) or per semantic item (item number, outcome, files, message). It is the aggregate's `evidence`;
  the aggregate's `value` is the tally as text (`16 of 16 items hold`).
- Each non-PASS command or item is one finding: `record.schema:product:<locus>` or
  `record.semantic:item-<n>:<locus>`. `<locus>` is the file path, or the offending ID for an item that spans files,
  lower-cased, with every character outside `a-z0-9_` as `-` and every leading character outside `a-z` dropped
  (`hw-product-yaml`, `req-003`, `cad-work-u1-brief-yaml`). A FAIL finding names the file, JSON
  path and message.
- A command that did not run is UNKNOWN `NOT_RUN`, never PASS.
