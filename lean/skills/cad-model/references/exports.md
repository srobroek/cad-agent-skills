# Exports, readback and geometry checks

Owner: cad-model. Build programs, the version build that cad-flow `build-version` runs, export formats, readback,
and the methods of `geometry.build`, `geometry.readback`, `geometry.export`, `geometry.identity`,
`geometry.manufacture` and `preview.render`. Anchors, producers and the result envelope: cad-verify
references/families.md and references/results.md. Environments, frames, components and witnesses:
references/model-contract.md. Kernel calls: references/cadquery.md and references/build123d.md. Paths: cad-record
`layout.json`.

## Build programs

Every build program, a unit `build.py` or a version `source/<kernel>/build.py`:

1. Takes its directories as absolute arguments, reads only its inputs and writes only below its output and work
   directories.
2. Loads each `model.py` by file path, reads `KERNEL` by parsing, and stops when it differs from part.yaml
   `model.kernel` or from the program's own environment.
3. Builds each definition once, PNs in bytewise order, and `build_manufacture` once for each `DIFFERENT` part.
4. Applies references/model-contract.md: finite `PARAMS`, valid shapes, positive volume, component binding, Boolean
   bounds and functional rounds.
5. Writes the exports (Export formats), then `signature.json` and the witnesses (Signatures and witnesses).
6. Exits non-zero with the PN, model line and reason on stderr on any failure.

## Signatures and witnesses

The work directory receives:

- `signature.json`, keys sorted: per PN and state (`definition` or the state name) `kernel`, `valid`, `solids`,
  `volume_mm3`, `bbox_mm` (min and max corners) and `components` (per component id its solids, volume and bbox);
  per occurrence its PN and world `translation` and `rotation` at the nominal pose; `fallbacks`, every cosmetic
  round left sharp.
- `features/<PN>.json` from `features`, and `features/<PN>.manufacture.json` from `manufacture_features`.

Checks and drawing programs reproduce these by rerunning the version programs from `hw/versions/vN/source/` on
`hw/versions/vN/record/` into their own scratch, and cite them as artifacts (role `DATA`); never from another
unit's or the lead's scratch.

## Unit build

For a `cad-part-designer` or `cad-concept-designer` unit:

1. Write each model: the part designer at `.cad-work/<unit>/output/hw/parts/<PN>/model.py`; the concept designer's
   coarse model inside `build.py`.
2. Write `.cad-work/<unit>/build.py` (layout `scratch.unit_build`). It reads the brief inputs under
   `.cad-work/<unit>/input/hw/` and the unit's own models, and takes two arguments: the export directory and the
   work directory. Lock it (`.cad-work/<unit>/build.py.lock`, layout `scratch.unit_build_lock`), then run it with
   stdout and stderr in `.cad-work/<unit>/output/logs/`, in the `uv run` form of
   [cad-flow agent-roles — uv commands](../../cad-flow/references/agent-roles.md#uv-commands) with these arguments:

   ```text
   <root>/.cad-work/<unit>/output/build/version <root>/.cad-work/<unit>/output/build/work
   ```

   A concept designer builds in the build123d environment and exports directly to
   `.cad-work/<unit>/output/hw/concepts/<variant>/geometry/`: `<variant>.step` (role `DEFINITION`), `<variant>.glb`
   (`VISUALIZATION`) and `<variant>.svg` (`PREVIEW`), as the Export formats rules state.
3. Write `.cad-work/<unit>/check.py` (layout `scratch.unit_check`, lock `scratch.unit_check_lock`) in the build123d
   environment. It runs the method below for each `expected.checks` entry on the unit's exports, writes
   `.cad-work/<unit>/output/checks/<check>/result.json` bound `UNIT`, and never imports `build.py` or a model.
4. A failed build, or a FAIL or UNKNOWN own check, delivers MAJOR with `ready: false` (cad-flow
   references/agent-roles.md). Part exports stay unit evidence in scratch; only records, models and retained
   sources are delivered.

## Version build

Run by the lead in cad-flow `build-version`. The stage tree mirrors the published version:

| Stage path | Published at | Content |
|---|---|---|
| `.cad-work/stage-vN/version/record/<p>` | `hw/versions/vN/record/<p>` | a byte copy of each `hw/<p>` in `.cad-work/stage-vN/input-inventory.txt` |
| `.cad-work/stage-vN/version/source/<kernel>/build.py` and `build.py.lock` | `hw/versions/vN/source/<kernel>/` | one version program per kernel, and its lock |
| `.cad-work/stage-vN/version/{definition,manufacturing,previews}/**` | `hw/versions/vN/{definition,manufacturing,previews}/**` | the exports |
| `.cad-work/stage-vN/build/` | not published | signature, witnesses, stdout and stderr; scratch diagnostics |

1. Snapshot: copy each inventoried file; each copy's sha256 equals its inventory line, else `SOURCE_MUTATION`.
2. A stage whose policy lists no `geometry.build` stops here: `build`, `runtime` and `geometry` are `NOT_APPLICABLE`
   with the cad-flow reason and `manufacturing` is empty.
3. Write `source/cadquery/build.py` when any PN's AUTHORED or PROXY model is CadQuery, and always
   `source/build123d/build.py`. Each kernel program writes the part STEP, state STEP, component files and STL of its
   own PNs. The build123d program also writes each SUPPLIER part STEP from its retained file, then imports every part
   STEP and writes the assembly STEP, the GLB, the previews and the flat profiles. Each takes the record, version and
   work directories as arguments.
4. Lock each program, then run `source/cadquery/build.py` first and `source/build123d/build.py` second, each with
   its own work directory, in the `uv run` form of
   [cad-flow agent-roles — uv commands](../../cad-flow/references/agent-roles.md#uv-commands) on the program
   `<root>/.cad-work/stage-vN/version/source/<kernel>/build.py` and these arguments:

   ```text
   <root>/.cad-work/stage-vN/version/record <root>/.cad-work/stage-vN/version <root>/.cad-work/stage-vN/build/<kernel>
   ```

5. The build123d program reads `assembly.step` back after writing it and reports its solid count, instance count
   and validity for manifest `geometry.readback`, with the import call as `method`.
6. Publishing is cad-flow's ([build-version](../../cad-flow/references/stages.md#build-version) step 3): exactly the files the
   manifest lists, one by one, to the same path below `hw/versions/vN/`, and `manifest.json` last. A staged file the manifest does not
   list, such as a `__pycache__/` entry, is never published.

Manifest fields this build supplies (prose root `version-manifest`, cad-record references/prose-contract.md):

- `record` and `source_artifacts`: the snapshot files, retained sources in `source_artifacts`; `build`: the programs
  as `source` and their locks as `locks`.
- `runtime.environments[]`: one per kernel with `kernel`, `python`, `packages` from its lock and `lock`; `platform`;
  `commands` (argv, cwd and exit code; the build logs stay scratch diagnostics, so no `stdout` or `stderr`).
- `geometry`: `units: mm`, `frame: RIGHT_HANDED_Z_UP`, every export except `manufacturing.yaml` in `files` with
  `role`, `format` and `PN`, `component` and `state` where they apply; `occurrences` with world `translation` and
  `rotation` at the nominal pose and `step_name`; `export_tolerance`; `readback`.
- `manufacturing[]`: per built MAKE or MADE_TO_DRAWING PN its `relationship`, `state`, `components`, export
  `orientation` and `files`.
- An occurrence of a PN whose model is `NONE` has no STEP instance and no manifest occurrence.

## Export formats

| Output (layout id) | When | Rule |
|---|---|---|
| `definition/parts/<PN>.step` (`version.definition_part`) | the PN's model is AUTHORED, PROXY or SUPPLIER | native frame, mm, the part's solids in a product named `<PN>` (CadQuery adds an unnamed root product); role `DEFINITION`, `PROXY` for a PROXY model; a SUPPLIER file is imported and written again in mm |
| `definition/assembly.step` (`version.definition_step`) | the stage lists `geometry.build` | nominal pose, one instance per occurrence named `<PN>@<occurrence>` (its `step_name`), never fused |
| `definition/assembly.glb` (`version.definition_glb`) | as the assembly STEP | binary glTF 2.0 from the build123d environment; role `VISUALIZATION` |
| `manufacturing/<PN>/part.step` (`version.manufacturing_step`) | MAKE or MADE_TO_DRAWING, from PARTS on | the manufacturing state in its export pose, in a product named `<PN>`; role `MANUFACTURE` |
| `manufacturing/<PN>/part.stl` (`version.manufacturing_stl`) | the route has an FDM or RESIN operation | binary, mm, the state shape meshed at absolute `export_tolerance` deflections; role `MANUFACTURE` |
| `manufacturing/<PN>/components/<COMP>/part.{step,stl}` (`version.component_*`) | the part lists components with separate routes or states | each component's own solids in its own export pose |
| `manufacturing/<PN>/flat.{dxf,svg}` (`version.manufacturing_flat_*`) | a LASER, WATERJET or PLASMA cut, or a SHEET_FORMING blank | the flat state's profile in its XY plane, mm, closed loops; role `PROFILE` |
| `manufacturing/<PN>/manufacturing.yaml` (`version.manufacturing_summary`) | as the state STEP | the manifest `manufacturing[]` entry of the PN, its `files` omitting this file |
| `previews/assembly.svg`, `previews/parts/<PN>.svg` (`version.preview_*`) | the stage lists `geometry.build` | visible edges from a stated view direction, one layer per occurrence named by its id; role `PREVIEW` |

- Export pose: an additive state is rotated by the APPROVED `orientation` of its first FDM or RESIN operation, then
  translated so its minimum Z is 0. A PROPOSED orientation gives no manufacturing export. Every other state keeps its
  native frame.
- `export_tolerance`: 0.05 mm linear and 0.1 rad angular deflection (cad-agent v0.1.1 defaults), finer when a
  check-plan criterion or the route's capability profile needs it; `glb: METRE_Y_UP`.
- Both kernels write STEP as AP214 (`AUTOMOTIVE_DESIGN`) with `SI_UNIT(.MILLI.,.METRE.)`; never claim AP242.
- Names carry no spaces: build123d `import_step` replaces spaces in labels with `_`.

## Lite build

A lite build ([cad-record record — Build mode](../../cad-record/references/record.md#build-mode)) builds at LAYOUT only the parts
references/model-contract.md "Lite model" describes. The Version build runs unchanged. Every lite PN is AUTHORED with no manufacturing
state, so the exports are exactly `definition/parts/<PN>.step` per PN, `definition/assembly.step`, `definition/assembly.glb`,
`previews/assembly.svg` and `previews/parts/<PN>.svg`. Every STEP is AP214 as Export formats state, the version has no `manufacturing/`
file, and the manifest `manufacturing` is `[]`. Each check heading below has a Lite paragraph for the checker's VERSION check; a
producer's UNIT checks keep the method above it. [FreeCAD readback](#freecad-readback) is a step of lite `geometry.readback`, never a
family of its own.

## Readback

A separate program in the build123d environment reads the exported files; it never imports the producing program or
a model, except the lite `geometry.build` harness that shares its `check.py`. Compare against the signature of a fresh rebuild:

- STEP: the header length unit is `SI_UNIT(.MILLI.,.METRE.)`; every solid is valid; the solid count equals the
  signature's; volume within 1e-6 relative; bbox within 1e-3 mm; instance labels equal the manifest `step_name`s; each
  instance transform equals its occurrence within 1e-6 mm and 1e-9 per quaternion component (sign-normalized).
- Witnesses: each patch point lies within 1e-3 mm of a face of its `support` type, with that face's normal within
  1e-3 rad.
- STL: binary; coordinates finite; after welding by position and dropping triangles of 1e-12 mm² or less, a closed
  2-manifold with consistent winding and positive signed volume; shell count equals the STEP solid count; volume
  within area × (deflection + 1e-5 mm) of the STEP's. Nothing is repaired.
- GLB: magic `glTF`, version 2; one node per occurrence named its `step_name`; composed node transforms map the STEP
  world bbox, scaled by 1e-3, through (x, y, z) → (x, z, −y) within 1e-6 m.
- DXF and SVG: parse; every loop closed; outer-loop bbox and loop count equal the flat state's profile face within
  1e-3 mm.

## geometry.build

1. In fresh scratch, run every version program from `hw/versions/vN/source/` in the `uv run` form of
   [cad-flow agent-roles — uv commands](../../cad-flow/references/agent-roles.md#uv-commands) on `hw/versions/vN/record/`,
   CadQuery first; for a UNIT check, rerun the unit's `build.py` into new directories.
2. Read the fresh exports back and compare them with the published exports and the signature tolerances of Readback.

FAIL when a program exits non-zero; a shape is invalid or has no positive volume; a component binding or Boolean
bound fails; a functional round fails; or a fresh signature differs from the published geometry beyond tolerance.
UNKNOWN `MISSING_INPUT` when a program, lock or snapshot file is absent; `TIMEOUT` when a run exceeds its limit.

Lite: the checker runs no version program and no producer `build.py`. In fresh scratch its own harness loads each snapshot
`hw/versions/vN/record/parts/<PN>/model.py` per references/model-contract.md (`KERNEL` by parsing, then `build` and `features` on a copy
of `PARAMS`). It places each occurrence from the snapshot `assembly.yaml` and writes the signature per Signatures and witnesses. The
harness is `check.py` for build123d models. When the version has CadQuery models, the checker's own `.cad-work/<unit>/build.py` and lock
in the CadQuery environment rebuild them first. That run is listed in `commands` with `artifacts/build-stdout.txt` and
`artifacts/build-stderr.txt` (LOG), writes `artifacts/build-signature.json` (DATA) for `check.py`, and is retained as `source/build.py`.
PASS only when every harness run exits 0 and finishes, and every expected solid is non-null, valid (`BRepCheck_Analyzer`), closed (every
shell `BRep_Tool.IsClosed_s`) and of finite positive volume. Each component's witness and `multiplicity` must also bind, and the fresh
signature must match the published STEP within the Readback tolerances: signatures match, never STEP bytes or header timestamps. FAIL and
UNKNOWN as above; an open shell is FAIL.

## geometry.readback

Apply Readback to every published STEP, STL, GLB, DXF and SVG of the subject against the fresh signature and
witnesses. `@assembly` covers the assembly STEP and GLB; `@<PN>` the part's STEP, manufacturing files and witnesses.

FAIL when any Readback rule fails. UNKNOWN `MISSING_INPUT` when a file the manifest lists is absent.

Lite: Readback applies to the published part and assembly STEP and the GLB of the subject, against the lite `geometry.build`
signature. Every measured value comes from the exported files, never from the harness's in-memory shapes, even inside one `check.py`.
The lite method adds:

1. Closure: every STEP solid is closed, as in lite `geometry.build`.
2. Owner criteria, in `geometry.readback@assembly`: measure each `geometry.readback` criterion mapping
   ([cad-verify check-plan — Lite build](../../cad-verify/references/check-plan.md#lite-build)) on the exported assembly STEP in the
   assembly frame. Bounding box and envelope come from the solids, counts from the instances and solids, and a feature dimension from the
   faces the feature's witnesses locate. Compare each with the criterion's comparator and value. Write one row per mapping to
   `artifacts/criteria.json` (DATA): mapping id, requirement, criterion, measured value with units, outcome.
3. [FreeCAD readback](#freecad-readback) of every part and assembly STEP of the subject.

PASS only when Readback, every criteria row and FreeCAD readback pass. FAIL when any of them fails: a wrong scale, count, volume or bounding
box is FAIL however well the signature and FreeCAD agree. UNKNOWN `REQUIRED_UNKNOWN` when a mapped criterion is UNKNOWN, and the FreeCAD
readback codes for the whole obligation.

## FreeCAD readback

The required step of a lite checker's VERSION `geometry.readback`. The checker proves that each published part and assembly STEP of the
subject opens in the installed FreeCAD, at the versions it records. A producer's UNIT readback omits this step and claims nothing about
FreeCAD. Nothing ships; every file below is the product project's.

1. Executable: the brief's `resources.tools` entry `freecad` (its `argv` from cad-flow preflight). When the entry is absent or its
   executable cannot be started, the obligation is UNKNOWN `NOT_RUN` with reason
   `FreeCAD executable unavailable; STEP compatibility not tested`, even when every other rule passed. It is never NOT_APPLICABLE, PASS, a
   waiver or another importer.
2. Helper: `check.py` writes `.cad-work/<unit>/tmp/<check>/freecad-readback.FCScript` and never edits it after the run. FreeCAD runs an
   `.FCScript` as `__main__`, while it imports a `.py` argument as a module and skips its main guard. The helper holds as literals the
   absolute paths of the STEP copies, the world witness points of each file and the output path. It imports only FreeCAD's own `FreeCAD`,
   `Part` and the standard library, so FreeCAD's bundled Python and OCCT never mix with a kernel environment. It makes no fix, sew, refine,
   scale or other healing call.
3. Per input the helper writes one entry keyed by the input's published `hw/` path (a literal beside its copy path), with the sha256 of the
   bytes it read; the copy path appears only in the helper source, never in the output. It then `Part.read`s the file and, for each of
   `shape.Solids` (assembly instances once each, never a container compound), writes `isValid()`, `isClosed()`, `Volume` in mm³, `BoundBox`
   minimum and maximum in mm and the witness ids it contains (`isInside(point, 1e-6, True)`). An import exception is recorded per file. It
   also writes the environment facts `FreeCAD.Version()`, `Part.OCC_VERSION`, `sys.version`, `FreeCAD.getHomePath()` and `sys.executable`
   (where it ran, never an evidence path), then `complete: true` last, into `artifacts/freecad-readback.json` (DATA), which `check.py`
   deletes before the run.
4. Run: `check.py` creates `<tmp>/<check>/freecad-home`, then runs the `argv` plus the helper's absolute path. The run has stdin closed, cwd
   `<tmp>/<check>/`, an environment of only `HOME` and `FREECAD_USER_HOME` (both that home) and `TMPDIR`, and a timeout; FreeCAD 1.1.4
   exits 101 ("Unknown runtime error occurred while initializing FreeCAD") without `HOME`. stdout and stderr go to
   `artifacts/freecad-stdout.txt` and `artifacts/freecad-stderr.txt` (LOG). The argv with the resolved absolute executable, cwd,
   environment keys, timeout and exit code go to `artifacts/freecad-run.json` (DATA). The result's `source_files` lists
   `source/freecad-readback.FCScript`, which cad-flow retains from `tmp/<check>/` at merge.
5. Compare in `check.py`. The output is complete, the exit code is 0, each input's sha256 equals its brief inventory entry, and the home
   path contains the executable. Each file's solid count equals the expected instance-solid count of the lite `geometry.build` signature.
   Each solid matches one expected component by the witness it contains, or a `multiplicity` group by volume and bounding box, never by
   index. Each matched solid is valid, closed and of positive volume, with volume within 1e-6 relative and bounding box within 1e-3 mm of
   the signature.

```python
shape = Part.read(step_path)  # inside the FCScript; no fix, sew, refine or scale
rows = [{"valid": s.isValid(), "closed": s.isClosed(), "volume_mm3": s.Volume,
         "bbox_mm": [[s.BoundBox.XMin, s.BoundBox.YMin, s.BoundBox.ZMin],
                     [s.BoundBox.XMax, s.BoundBox.YMax, s.BoundBox.ZMax]]} for s in shape.Solids]
```

PASS when step 5 holds for every file. FAIL when a file does not import, a count, validity, closure, volume or bounding box differs, or a
solid matches no or several components. UNKNOWN `NOT_RUN` per step 1. UNKNOWN `INCOMPLETE` when the output is missing, has no
`complete: true`, names other bytes or was not deleted before the run, or the exit code is non-zero. `TIMEOUT` at the limit. A banner or
exit code 0 alone is never evidence. The PASS holds only for the recorded FreeCAD, OCCT and Python versions; it says nothing about other
FreeCAD versions or physical use. After FreeCAD becomes available, unchanged inputs rerun as a new attempt; a changed input or method is a
new version.

## geometry.export

1. Derive the expected files from the Export formats conditions on the snapshot records.
2. Compare with the version's `definition/`, `manufacturing/` and `previews/` trees and the manifest `files`.

FAIL when an expected file is missing, an unexpected file is present, a sha256 differs from the manifest, a STEP
declares other units, an STL is ASCII or was meshed coarser than `export_tolerance`, or the GLB is not METRE_Y_UP.

Lite: the expected files are those of [Lite build](#lite-build). FAIL also when any `manufacturing/` file exists, a STEP's `FILE_SCHEMA` is
not `AUTOMOTIVE_DESIGN` (AP214), or a STEP does not re-import with build123d `import_step`. A delivered STEP is labelled AP214, never
AP242.

## geometry.identity

For each component of the PN, in the definition and in the manufacturing state:

1. Its witness lies inside exactly one solid, or `multiplicity` congruent solids; every solid binds once; no two
   solids share more than 1e-3 mm³; a component file holds exactly its own solids.
2. Every semantic feature the snapshot names for this PN or component (part.yaml critical surfaces, route targets,
   inspections, drawing characteristics and locators; interfaces.yaml members and datums) has a witness that passes
   Readback.

FAIL when a solid is unbound, ambiguous or claimed twice, or a named feature has no witness or a witness is off its
surface. UNKNOWN `MISSING_INPUT` when the part's components are not declared.

Lite: step 2 also covers each feature a `geometry.identity` criterion mapping names by its criterion `property`
([cad-verify check-plan — Lite build](../../cad-verify/references/check-plan.md#lite-build)). The feature belongs to the one PN whose fresh
`features` carry a witness of that id, and that witness must pass Readback. Write one row per mapping to `artifacts/criteria.json` (DATA):
mapping id, requirement, feature id, PN, witness found and on its surface, outcome. A mapped feature that no PN witnesses, or that two PNs
witness, makes every `geometry.identity` outcome FAIL. Identity infers no material or mass.

## geometry.manufacture

For each MAKE or MADE_TO_DRAWING PN:

1. `SAME`: the state STEP equals the definition in solid count and volume within 1e-6 relative.
2. `DIFFERENT`: `build_manufacture` exists, the state is named, `manufacture.decision` is APPROVED, solid counts are
   equal and the volume difference is within `volume_tolerance`; list the `manufacture_features` correspondence.
3. Export pose: a proper rotation (orthonormal, determinant +1 within 1e-9); an additive state carries the APPROVED
   orientation and rests on Z 0 within 1e-6 mm; any other state is at identity.
4. A route that needs a flat profile has `flat.dxf` and `flat.svg` matching the state's profile face.

FAIL when a rule fails. UNKNOWN `REQUIRED_UNKNOWN` when the orientation is PROPOSED or the state decision is not
APPROVED. Limitations: equal volume does not prove the same features or material; process feasibility is cad-verify's
`process.*`.

## preview.render

`@assembly` and `@<PN>`: each preview SVG exists, parses, and has a non-empty layer for every occurrence it shows.
`@product` at CONCEPT: each variant's `PREVIEW` geometry file parses and is non-empty.

FAIL when a preview is missing, unparseable or has an empty layer. Limitations: a preview is review material, never
verification of fit, motion or manufacture.

Lite: `@assembly` and `@<PN>` only. Each preview is also listed in the version manifest `files` with its sha256 and role `PREVIEW`. The
lead shows it to the owner at RELEASE ([cad-release](../../cad-release/references/release.md#lite-release)), and a result never claims it
was shown.

## For drawings

cad-release drawing programs read `definition/parts/<PN>.step` or `manufacturing/<PN>/part.step` with build123d
`import_step` in the build123d environment, and take feature locations from witnesses reproduced per "Signatures and
witnesses"; drawn values are measured on that STEP.
