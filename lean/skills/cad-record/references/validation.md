# Validation

Owner: cad-record. Two layers: the published `check-jsonschema` CLI proves the shape of the ten schema roots; a
model-written semantic check program proves what shape cannot, and the shape of the 17 prose roots
(references/prose-contract.md). Both report as check results (`record.schema`, `record.semantic`).

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
command runs on them.

A snapshot under `hw/versions/vN/record/` validates against the root, or conforms to the prose root, of the live
file it copies.

## Semantic check

JSON Schema does not prove items 1–16; item 17 stands in for the schema of each prose root. The lead or checker
writes its own program under `.cad-work/<unit>/` with an explicit uv script lock, runs it against the inventoried
input, and publishes a `record.semantic` check result; an independent checker reads that program before its result
counts.

The program reads the inventoried `hw/` files, this skill's schemas, `layout.json`, the stage policy and the
session statements the DRs hash, all named in the brief ([cad-flow worker brief](../../cad-flow/references/worker-brief.md)).
An item over a collection the record does not have yet (assembly occurrences before LAYOUT, deliveries, versions,
BOM, verification index, runtime) holds, its item-table row saying `no instance`. An input the brief lacks makes the
items that need it UNKNOWN `MISSING_INPUT` and is reported as a brief defect.

1. Finite numbers: reject NaN and ±infinity anywhere, including YAML `.nan`, `.inf` and `-.inf`, before any
   other use; serialize JSON with non-finite numbers refused.
2. Uniqueness: every ID inside its collection and across files (PN, REQ, IF, DR, Q, ASM, PRF, case, check,
   variant, occurrence, joint, component, operation).
3. References: every cited ID, profile, case, check, obligation and path resolves to an existing entry.
4. Assembly tree: one root, every parent exists, no cycle; joints connect existing occurrences.
5. Order and units: `lower <= upper`, BETWEEN bounds ordered, joint limits ordered, each unit matches the
   property's dimension; quaternions have unit norm within 1e-9 and axes are non-zero.
6. Hashes: every recorded sha256 equals the bytes on disk; a retained source's directory name equals its
   sha256; inventories exclude themselves and every downstream file (references/record.md hash graph).
7. Freshness: a result's inputs, cases and check plan equal the version snapshot; calibrations and waivers
   are unexpired at the verdict time; a changed input makes the old evidence stale (UNKNOWN, `STALE`).
8. Coverage: a result's outcomes cover exactly its `expected_obligations`; a stage verdict's `expected`
   equals the due obligations the stage policy enumerates; future facets are `PLANNED`.
9. Chain and authority: predecessor verdict and gate DR exist and bind; vN is unused and above every
   attempt; approvals are USER or AUTHORITY statements with a transcript hash; PRODUCTION and
   SAFETY_CRITICAL scope names a competent authority.
10. Waivers in a stage verdict:
    - Existence: each `WAIVED` result's `WV-###` is the `waiver.id` of an APPROVED WAIVER DR that no later
      DR rejects or supersedes, and the verdict's `waivers` list equals the set of waivers its results cite.
    - Scope coverage: the waiver's obligation and subject cover every expected obligation of that check that
      is not PASS; the check's own `result.json` result is FAIL or UNKNOWN and equals `retained_outcome`; no
      covered obligation is safety or regulatory, and neither its requirement nor its check plan entry has
      `waiver_policy: FORBIDDEN`.
    - Expiry: `expires` is after the verdict `time`, or the `until_change` condition has not occurred.
    - Every other result's `result` equals the `result` in its `result.json`.
11. Digests: every `input_inventory` and `boundary` digest recomputes exactly by
    references/record.md#inventory-digest; a REVIEW implementation's `inspected` hashes match the bytes read,
    and `inspected` never lists the review's own `result.json` or `bundle-manifest.json`.
12. Immutability and layout: published DRs, verdicts, manifests and results match their recorded hashes;
    every `hw/` file matches a layout entry; no evidence cites `.cad-work/`.
13. Derived files: BOM rows sorted by path then component, verification rows by requirement, subject and
    facet; each CSV equals its YAML authority.
14. Bootstrap: after GROUND, `product.yaml` `id` and `title` are no longer the template values.
15. Version runtime: each `runtime.environments[].lock` equals one `build.locks` entry (path and sha256), no
    two environments share a lock, and every built PN's `model.kernel` has the environment of that kernel.
16. Self-referencing evidence: no verification row cites a `verification.index` result file, and no variant,
    integrity result or index carries the digest of a file that hashes it (references/record.md hash graph).
17. Prose roots: each instance of a [prose root](prose-contract.md) conforms to its table, one item-table row per file:
    - every required field is present and no unlisted field appears, at every level;
    - constants, enums, array minimums, uniqueness and every `when`, `iff` and `never` condition and listed rule hold;
    - each typed value validates in-process against its named `common.schema.json` or `part.schema.json` `$defs`
      entry, and each `root:<name>` item (material library `profiles[]`, BOM `rows[]`) against that schema root,
      resolving `$ref`s from `<skill-dir>/schemas/`;
    - the stage policy's `order` runs 1–13 in stage-enum order, GROUND's `predecessor` is null and every other
      stage's `predecessor` is the stage before it.

## Reporting

`record.schema@product` and `record.semantic@product` each get one aggregate outcome, the only outcome of their
check result ([cad-verify results — Outcomes](../../cad-verify/references/results.md#outcomes)):

| Aggregate | When |
|---|---|
| `PASS` | every schema command exits 0; every semantic item holds |
| `FAIL` | any command exits non-zero, or any item is violated |
| `UNKNOWN` | otherwise, when a command did not run or an item could not be assessed: the `unknown_code` all those share, else `INCOMPLETE` |

- Item table: `artifacts/items.json` in the check bundle, role `REPORT`; one row per command (file, root, exit
  code, stderr) or per semantic item (item number, outcome, files, message), item 17 one row per prose-root file.
  It is the aggregate's `evidence`; the aggregate's `value` is the tally as text (`17 of 17 items hold`).
- Each non-PASS command or item is one finding: `record.schema:product:<locus>` or
  `record.semantic:item-<n>:<locus>`. `<locus>` is the file path lower-cased with every character outside `a-z0-9_`
  as `-` (`hw-product-yaml`), or the offending ID for an item that spans files. A FAIL finding names the file, JSON
  path and message.
- A command that did not run is UNKNOWN `NOT_RUN`, never PASS.
