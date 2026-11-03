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
| version | `v[1-9][0-9]*` | one stage attempt, see Versions |
| check attempt | `a[1-9][0-9]*` | under `hw/checks/vN/<check>/` |
| case, variant, check | `[a-z][a-z0-9_-]*` | |
| component | `[a-z][a-z0-9_]*` | one disconnected solid |
| feature, region, operation, occurrence, joint | `[a-z][a-z0-9_]*` | semantic names |
| unit | `[a-z][a-z0-9_-]*`, not `lead`, not `stage-*` | `.cad-work/<unit>/` |
| obligation | `<family>@<anchor>` | anchor grammar in references/contract-index.md |
| finding | `<family>:<anchor>:<feature-or-operation>` | stable, never positional |

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
| IMMUTABLE | written once; any change is a new attempt, version, DR or REV |
| APPEND_ONLY | new files only (decisions) |
| DERIVED | regenerated from one named authority (CSV views); never edited, never evidence |
| NOTES | `hw/docs/**`; human notes, no gate credit, no scripts, results or geometry |
| SCRATCH | `.cad-work/**`; never durable evidence; nothing counts until published under `hw/` |

## Acyclic hash graph

No input contains downstream result bytes or its own future hash.

1. Working input inventory: record, models, params, cases, retained sources and policy. Unit builds and
   UNIT-bound checks bind the brief and this inventory, never a future vN digest.
2. Version manifest (`hw/versions/vN/manifest.json`), written after geometry build and readback: record
   snapshots, build source and locks, runtime, geometry and manufacturing inventories, units, transforms and
   export tolerance. It excludes itself, check results, the stage verdict, later decisions and every
   drawing, BOM or verification output that embeds its digest.
3. Check result `binding`: `UNIT` (unit, brief_sha256, input_inventory_sha256) or `VERSION` (version,
   version_manifest_sha256). Its artifact and implementation inventories exclude `result.json` and
   `bundle-manifest.json`. A UNIT result is never integrated stage evidence by itself.
4. Bundle manifest: inventories the result, source, lock, artifact and log bytes and the input identity;
   excludes itself. Drawing, BOM and verification bundles are separate output bundles bound the same way.
5. Stage verdict (`hw/versions/vN/stage-verdict.json`): written once after every required check; names
   the expected obligations, accepted result and bundle hashes, predecessor and its approval, version
   manifest and policy hash. No self hash.
6. Gate DR after a PASS verdict binds that verdict (`binding.kind: STAGE`). Earlier selection, orientation,
   support, waiver, change and import DRs bind an immutable proposal (`PROPOSAL`) retained at
   `hw/decisions/proposals/DR-###/<name>` before the DR is written; scratch is never bound. Approve,
   reject and supersede are new DRs; an old DR's status derives from the DRs that refer to it.
7. A successor attempt names its approved predecessor; a retry names the accepted causal baseline and the
   superseded failure. Impact reopens dependents explicitly; no step of the chain is skipped.
8. Release: the PROPOSAL manifest (`hw/versions/vN/release-proposal.json`) lists the planned payload and
   excludes the not-yet-written approval and integrity result. The FINAL manifest adds approval, the
   package-integrity result and the full inventory, and excludes itself; its digest is reported outside it.

A checker may inspect scratch before the manifest exists, then emit its VERSION result against the identical
inventoried input. Whole-assembly checks rerun on the integrated version; never relabel a UNIT PASS.

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

## Template and bootstrap

- `assets/project-template/hw/` holds empty collections and explicit UNKNOWN facts with owners and the
  open questions `Q-001`…`Q-017`. It sets no printer, material, process, tolerance, regime, general
  tolerance, clearance, rigor or license.
- Copy it only when `test ! -e hw && test ! -L hw` succeeds; never overwrite, merge or follow a symlink.
  Then replace `id` and `title` with the owner's product name, and validate.
- The template is schema-valid with zero requirements. SPECIFY semantic completeness rejects a mandatory
  requirement without engineering acceptance.
