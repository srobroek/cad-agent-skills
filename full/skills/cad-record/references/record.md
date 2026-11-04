# Record contract

Owner: cad-record. `layout.json` (beside SKILL.md) is the path authority; the schemas are the field authority.
This file states the rules the schemas cannot express.

## Namespace and conventions

- `hw/product.yaml` declares `format: cad-agent-skills` and `record_version: 1`. This is not cad-agent v1;
  a cad-agent record of any version is imported (references/import.md), never read in place.
- Generated JSON and YAML documents declare `format: cad-agent-skills.<root>` and `schema_version: 1`.
  Authored record YAML (requirements, interfaces, assembly, check plan, parts, manufacturing, decisions)
  carries no header; product.yaml carries the record version for the project.
- Machine states, processes, service classes and results are CAPS. Exceptions: rigor `maker|pro|regulated`,
  kernel `cadquery|build123d`, unit symbols (`mm`, `deg_C`, `g_cm3`…) and quoted source text.
- New geometry is authored `PART_LOCAL`, millimetres, right-handed, +Z up. An imported part keeps
  `ASSEMBLY_ZERO` only with `model.import` naming its original and approving DR, until migrated by a new DR.
- Dates are `YYYY-MM-DD`, instants UTC `YYYY-MM-DDThh:mm:ssZ`, both quoted in YAML.
- Paths inside records are project-relative (`hw/…` or `.cad-work/…`): no `..`, `.`, empty segment,
  trailing slash, backslash or control character. Paths a human reads in a report are absolute.

## IDs

| Family | Pattern | Notes |
|---|---|---|
| PN | `[A-Z]{2,5}-[0-9]{4}` | never reused for a different part |
| part revision | `X[1-9][0-9]*` draft, `[A-HJ-NPRTUVWY]` released | only a changed part's revision increases |
| release REV | `[A-HJ-NPRTUVWY]` | package revision, distinct from part revisions; a sealed REV is never reused |
| REQ, IF, DR | `REQ-###`, `IF-###`, `DR-###` | Q, ASM, PRF use `Q-###`, `ASM-###`, `PRF-###` |
| waiver | `WV-###` | `waiver.id` inside its WAIVER DR; stage verdicts cite it |
| version | `v[1-9][0-9]*` | one stage attempt, see Versions |
| check attempt | `a[1-9][0-9]*` | under `hw/checks/vN/<check>/` |
| case, variant, check | `[a-z][a-z0-9_-]*` | |
| component | `[a-z][a-z0-9_]*` | one disconnected solid |
| feature, region, operation, occurrence, joint | `[a-z][a-z0-9_]*` | semantic names |
| unit | `[a-z][a-z0-9_-]*`, not `lead`, not `stage-*` | `.cad-work/<unit>/` |
| obligation | `<family>@<anchor>` | anchor grammar in references/contract-index.md |
| finding | `<family>:<anchor>:<feature-or-operation>`; `record.semantic:item-<n>:<locus>` | stable, never positional; locus and item per references/validation.md#reporting |

NOT Derive identity from a timestamp, array index, random value or OCCT face/edge/solid index.
NOT Promote a part revision after FREEZE; promotion happens before the final checks.

## Versions

- `vN` is one stage attempt, including pre-CAD stages. Allocate the next integer above every existing
  attempt and sealed label; never reuse one.
- A stage without geometry records `geometry: {status: NOT_APPLICABLE, reason}`; never a placeholder STEP.
- A changed input, case, method or implementation, or a correction after a completed FAIL, consumes the
  next vN. An interrupted attempt with unchanged inputs resumes in place.
- No mutable "current stage", "latest" pointer or CSV approval exists. On resume, derive the chain from
  stage verdicts and gate DRs: each verdict names its predecessor and its gate DR binds it.

## Mutability

| Class | Rule |
|---|---|
| CONTROLLED | changed only by the lead through the stage workflow; after FREEZE only through a CHANGE DR |
| IMMUTABLE | written once; any change is a new attempt, version, DR or REV. An import's originals, legacy programs included, sit at `hw/imports/DR-###/original/**` with no gate credit and are never executed |
| APPEND_ONLY | new files only (decisions) |
| DERIVED | regenerated from one named authority (CSV views); never edited, never evidence |
| NOTES | `hw/docs/**`; human notes, no gate credit, no scripts, results or geometry |
| SCRATCH | `.cad-work/**`; never durable evidence; nothing counts until published under `hw/` |

## Acyclic hash graph

No input contains downstream result bytes or its own future hash.

1. Working input inventory: record, models, params, cases and retained sources under `hw/`, digested as in
   [Inventory digest](#inventory-digest). Unit builds and UNIT-bound checks bind the brief and this digest,
   never a future vN digest. The installed policy is named by path and plugin version, never hashed.
2. Version manifest (`hw/versions/vN/manifest.json`), written after geometry build and readback: record
   snapshots, build source and locks, runtime (one locked environment per kernel, each naming its lock),
   geometry and manufacturing inventories, units, transforms and
   export tolerance. It excludes itself, check results, the stage verdict, later decisions and every
   drawing, BOM or verification output that embeds its digest.
3. Check result `binding`: `UNIT` (unit, brief_sha256, input_inventory_sha256) or `VERSION` (version,
   version_manifest_sha256). Its artifact inventory and COMMAND sources exclude `result.json` and
   `bundle-manifest.json`; a REVIEW `inspected` list may name other, earlier checks' result and bundle
   files, never its own. A UNIT result is never integrated stage evidence by itself.
4. Bundle manifest: inventories the result, source, lock, artifact and log bytes and the input identity;
   excludes itself. Drawing, BOM and verification bundles are separate output bundles bound the same way.
   The verification index is the input of its `verification.index` check, so no row cites that check's
   obligation or result.
5. Stage verdict (`hw/versions/vN/stage-verdict.json`): written once after every required check; names
   the expected obligations, accepted result and bundle hashes, waivers, predecessor and its approval,
   version manifest, and the installed policy by path and plugin name and version. No self hash.
6. Gate DR after a PASS verdict binds that verdict (`binding.kind: STAGE`). Earlier selection, orientation,
   support, waiver, change and import DRs bind an immutable proposal (`PROPOSAL`) retained at
   `hw/decisions/proposals/DR-###/<name>` before the DR is written; scratch is never bound. Approve,
   reject and supersede are new DRs; an old DR's status derives from the DRs that refer to it.
7. A successor attempt names its approved predecessor; a retry names the accepted causal baseline and the
   superseded failure. Impact reopens dependents explicitly; no step of the chain is skipped.
8. Release: the PROPOSAL manifest (`hw/versions/vN/release-proposal.json`) lists the planned payload and
   names the frozen FREEZE version and verdict; it excludes the not-yet-written RELEASE readiness verdict,
   approval and integrity result, because the RELEASE checks and verdict hash the proposal. The RELEASE
   approval DR then binds both the proposal and that readiness verdict (`RELEASE_INVENTORY`). The FINAL
   manifest adds the readiness verdict, the approval, the package-integrity result and the full inventory,
   and excludes itself; its digest is reported outside it. The package-integrity result is hashed by the
   FINAL manifest, so it never hashes or inspects any `release-manifest.json`; it checks the staged payload
   against the proposal's planned inventory.
9. Worker outputs: a delivery's `outputs` hash each delivered file, so a delivered file (such as
   `variant.yaml`) never holds its delivery digest; it names its unit, role and brief digest, and the lead
   records the accepted delivery digest in `units.yaml`.

A checker may inspect scratch before the manifest exists, then emit its VERSION result against the identical
inventoried input. Whole-assembly checks rerun on the integrated version; never relabel a UNIT PASS.

## Inventory digest

One digest rule for `input_inventory.sha256` (worker brief), `input_inventory_sha256` (UNIT binding, worker
delivery), `boundary.before_sha256` / `boundary.after_sha256` (worker delivery) and `assessment.input_sha256`
(selection, a set digest that never lists `hw/manufacturing/selection.yaml` itself).

- Root: the project root, the directory that holds `hw/`. Every path is relative to it.
- Line: `<lowercase sha256 hex>  <path>` (two spaces) followed by LF: the `shasum -a 256` and `sha256sum` line.
- Path: POSIX `/` separators, no `./` prefix, safe-path rules (no backslash or control character, so no
  tool escapes a line).
- Order: bytewise by path, `LC_ALL=C sort`.
- Digest: sha256 of the concatenated lines. The empty set digests to
  `e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855`.
- Excluded always: `.git/**` and `.cad-work/**` (all scratch, including the unit's own). Only regular files
  count; symlinks are not followed and a symlink in scope is a defect for the semantic check; empty
  directories contribute nothing.
- Input inventory (set digest): exactly the paths in the brief's `input_inventory.files`, all under `hw/`;
  each entry's sha256 equals its line's hex.
- Boundary (tree digest): every regular file under the root except the exclusions, taken before the worker's
  first write and after its last write. A worker writes only in its own scratch, so the two must be equal;
  the lead recomputes the after digest on acceptance, and any difference is a boundary violation.

Tree digest, one line from the project root (the digest is the first field of the output):

```sh
find . -type f ! -path './.git/*' ! -path './.cad-work/*' | sed 's|^\./||' | LC_ALL=C sort | tr '\n' '\0' | xargs -0 -r shasum -a 256 | shasum -a 256
```

```sh
find . -type f ! -path './.git/*' ! -path './.cad-work/*' | sed 's|^\./||' | LC_ALL=C sort | tr '\n' '\0' | xargs -0 -r sha256sum | sha256sum
```

The first line is macOS (`shasum`), the second Linux (`sha256sum`). For a set digest, replace the `find … | sed …`
prefix with `LC_ALL=C sort <list>` over a file holding one listed path per line.

Worked example: `hw/product.yaml` holds `format: cad-agent-skills` + LF and `hw/parts/STD-0001/part.yaml` holds
`PN: STD-0001` + LF; `.git/HEAD` and `.cad-work/u1/x.txt` also exist and are excluded. The lines are:

```text
1822f76462de13a2333ba4c89f162447e6deba13e5c6015bdf32d4bcf314b17d  hw/parts/STD-0001/part.yaml
3eb0328bd1bb3221a0cd864c6f919614077c3128e46e6c13530533080467102b  hw/product.yaml
```

and the digest, from either command and from the set digest over those two paths, is
`9eaffd57ef4b45dece51bde758f86e30961a081fcc3dcc25b1a883e725909557`.

## Facts

Every fact is `KNOWN`, `UNKNOWN` or `NOT_APPLICABLE` (common.schema.json `fact`).

| Status / basis | Required |
|---|---|
| KNOWN | value (with units for a quantity), basis, sources |
| basis ESTIMATE | uncertainty |
| basis ASSUMED | an `ASM-###` assumption and its bounds |
| basis ALLOWABLE | authority, conditions, derivation; never from SUPPLIER_TYPICAL or ESTIMATE data |
| UNKNOWN | reason; owner and question when `consequential: true` |
| NOT_APPLICABLE | reason and `reviewed_by`; never an omission |

- A source names a URL with its access date or a retained artifact with its sha256 (datasheet, standard,
  calibration, test, supplier), plus document, revision and the conditions it actually states.
  Retained bytes live at `hw/manufacturing/sources/<sha256>/<original-name>` or `hw/parts/<PN>/sources/**`.
- A `USER` source is a statement of the owner or of the authority `product.yaml` names, made in a session:
  `origin` names them, `reference` the question it answers; the basis is `USER_DECLARED`. It is a sourced fact;
  nobody asks for its provenance or asks the question again.
- A test temperature, method, orientation or conditioning the supplier does not state stays UNKNOWN; a use
  that depends on it is blocked, never filled in.
- Density is never strength; typical is never allowable; a standard minimum stays typical_not_allowable.

## Service class and rigor

- Service class: APPEARANCE_CONCEPT, FIT_ASSEMBLY, FUNCTIONAL_DEMO, WORKING_PROTOTYPE, PRODUCTION,
  SAFETY_CRITICAL or UNKNOWN. A known class names its DR. A "prototype" request asks which class.
- Rigor sets documentation depth only: `maker` essential contracts and safety, `pro` detailed tradeoff,
  manufacture and inspection, `regulated` controlled authority, risk and standards traceability. Rigor
  never relaxes the evidence a service class needs.
- PRODUCTION and SAFETY_CRITICAL need a competent authority with supplied criteria, allowables and evidence.

## Template

- `assets/project-template/hw/` holds empty collections and explicit UNKNOWN facts with owners and the
  open questions `Q-001`…`Q-017`. It sets no printer, material, process, tolerance, regime, general
  tolerance, clearance, rigor or license.
- The template is schema-valid with zero requirements. SPECIFY semantic completeness rejects a mandatory
  requirement without engineering acceptance.

## record-bootstrap

Procedure `record-bootstrap`, run by cad-flow at GROUND from the project root. `<skill-dir>` is the absolute
cad-record directory from the brief or preflight.

1. When `test ! -e hw && test ! -L hw` succeeds, copy the template, then set `id` and `title` in
   `hw/product.yaml` from the owner's product name:
   `cp -R "<skill-dir>/assets/project-template/hw" hw`
   Never overwrite, merge into or follow an existing or dangling `hw`.
2. Otherwise, when `hw/product.yaml` declares `format: cad-agent-skills` and `record_version: 1`, keep the
   record unchanged.
3. Otherwise the record is legacy or foreign: run references/import.md (dry run, owner IMPORT DR, apply);
   no byte lands in `hw/` before that approval.
4. Run no check here: `record.schema` and `record.semantic` cover the whole record once per attempt in cad-flow
   `run-checks`. During intake, validate each changed file that has a schema root once per owner reply
   (references/validation.md#schema-validation). UNKNOWN facts with open questions are intake input, not failures.
5. Report the path taken (`COPIED`, `EXISTING` or `IMPORTED`) and every open question for GROUND intake.
