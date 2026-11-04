# Model contract

Owner: cad-model. The program contract of `hw/parts/<PN>/model.py` (cad-record layout `record.part_model`), the kernel
environments every build, check and drawing program runs in, frames and placement, components, feature witnesses and
the manufacturing state. Part fields are the cad-record `part` root. The product project owns every program; this
skill ships none.

## Top-level names

| Name | Required | Contract |
|---|---|---|
| `KERNEL` | yes | string literal `"cadquery"` or `"build123d"`, equal to part.yaml `model.kernel`; read by parsing the file, never by importing it |
| `PARAMS` | yes | dict literal of JSON values: lengths in mm, angles in degrees, counts as integers, lower_snake_case keys |
| `build(params)` | yes | pure function returning the definition (one solid, or one compound of the part's solids) in the native frame |
| `features(params, shape)` | when a record names a feature of this PN | the semantic feature witnesses of `shape`, below |
| `build_manufacture(params)` | exactly when part.yaml `manufacture.relationship` is `DIFFERENT` | the manufacturing state, below |
| `manufacture_features(params, shape)` | beside `build_manufacture` when an operation, inspection or drawing targets the state | witnesses of the state |

- Each function gets its own deep copy of `PARAMS`, returns a value, writes no file and changes no global state.
- Accepted returns: CadQuery `Shape`, or `Workplane` (every stack object joins one compound); build123d `Part`, `Solid`,
  `Compound`, or the `.part` of a `BuildPart`.
- Importing `model.py` runs no geometry; only the build program calls the functions.
- A concept designer's coarse model uses the same names inside its `build.py`; it has no `model.py` target.

## PARAMS authority

- Every design dimension is a `PARAMS` key; a literal elsewhere is only a unit-free factor or a kernel tolerance.
- The record holds criteria, sources and intent, never a second copy of a model value. A fit limit, clearance or wall
  in part.yaml or interfaces.yaml is a criterion the checks measure on the geometry, never read by the model.
- A key that realizes a sourced value (a fastener head size, a bearing bore) carries a comment naming the standard or
  datasheet and its retained file under `hw/parts/<PN>/sources/`.
- The definition always builds from the declared `PARAMS`. A check that builds at other values (a tolerance extreme)
  records each override in its result; a changed value is a `model.py` change and a new attempt.
- Reject a non-finite parameter (NaN, infinity) before any kernel call.

## Determinism

`model.py` and every program that builds, checks or draws:

- NOT read the clock, environment variables, the network, random numbers, the host name or the user;
- NOT iterate a set, a directory listing or kernel query results without a geometric sort key, and never pick by
  position from unsorted kernel output;
- NOT name geometry by an OCCT face, edge or solid index; select by surface type, axis, direction and position;
- read files only under `hw/parts/<PN>/sources/`, resolved from the model file's own directory, each listed in the
  input inventory with its sha256; never part.yaml, another PN or any other record file;
- draw text only from a font file retained under `hw/parts/<PN>/sources/` with its license file, passed by path;
  a family name alone or a host font is forbidden, and a missing font fails the build.
  Sole exception: drawing dimension labels, owned by
  [cad-release Drawing program](../../cad-release/references/drawings.md#drawing-program).

Determinism is the same signature at the declared tolerances (references/exports.md "Signatures and witnesses"),
never identical bytes; a STEP header timestamp may differ between runs.

## Kernels and environments

- One kernel per `model.py`. A model imports only its kernel, that environment's OCP, the standard library and a
  package its build program's header pins (bd_warehouse 0.4.0 serves build123d; cq_warehouse is not on PyPI).
- The two OCP builds clash: never import both kernels in one process, pass a shape between environments or add the
  other kernel's OCP. Geometry crosses environments only as STEP files; CadQuery parts are read in the build123d
  environment with `import_step` for assembly, readback and drawings.

| Kernel | Python | Pins (cad-agent v0.1.1 runners) |
|---|---|---|
| `cadquery` | 3.12 | `cadquery==2.8.0`, `cadquery-ocp==7.9.3.1.1`, `vtk==9.6.2` |
| `build123d` | 3.12 | `build123d==0.13.0`, `cadquery-ocp-novtk==8.0.1.1.0` |

Every program starts with an inline script header (PEP 723) pinning one row exactly, plus any extra pinned package:

```python
# /// script
# requires-python = "==3.12.*"
# dependencies = ["build123d==0.13.0", "cadquery-ocp-novtk==8.0.1.1.0"]
# ///
```

Lock, then run only locked (uv 0.13.6; the lock is written next to the script):

```sh
uv lock --script /abs/project/.cad-work/<unit>/build.py
uv run --locked --script /abs/project/.cad-work/<unit>/build.py <arguments>
```

- Worker locks are `.cad-work/<unit>/build.py.lock` and `.cad-work/<unit>/check.py.lock` (layout
  `scratch.unit_build_lock`, `scratch.unit_check_lock`). Always pass the absolute script path.
- A header the lock does not match fails the run; relock only after a deliberate header change, never to pass.
- `uv run --no-project --python 3.12 --with <pin> …` is a probe only (preflight); it freezes no transitive
  dependency and never produces evidence.
- Record per run: the resolved Python patch, every package and version the lock resolved, OS and architecture, argv,
  cwd, exit code and the stdout and stderr files.
- A cold environment took 170–320 s to import OCP in cad-agent v0.1.1 runs; never put a short timeout on a build.

## Frames and placement

- New geometry is `PART_LOCAL`: the part's own origin and axes, mm, right-handed, +Z up, with the origin on a
  functional datum (a joint axis, a seat face).
- An imported `ASSEMBLY_ZERO` part is already in assembly coordinates at the zero pose: one occurrence, identity
  placement, until a DR migrates it (cad-record references/record.md).
- Occurrence transform: world(occurrence) = world(parent) · placement, where placement rotates about the local origin
  by the unit quaternion `[w, x, y, z]`, then translates in mm, both in the parent frame. A top-level parent frame is
  the assembly frame.
- Joints: placements hold at joint value `zero`. Pose q moves the child by q − `zero` about (REVOLUTE, deg) or along
  (PRISMATIC, mm) the joint `axis` through `origin`, in the parent frame, after the placement. Definition exports use
  the nominal pose, every joint at `zero`.
- `build` never places its part; nothing guesses a frame from geometry.
- OCP `gp_Quaternion(x, y, z, w)` takes w last; the record stores `[w, x, y, z]`.
- GLB is visualization only: metres and +Y up, converted once by the exporter; never pre-rotate or pre-scale.

## Components

- Every disconnected solid of the definition and of the manufacturing state is a component (cad-record `component`),
  bound by a `witness`, an interior point at least 1e-3 mm inside its solid in that state's native frame, or by a
  `locator`, a semantic feature whose patch points lie on it.
- `multiplicity` n binds n congruent solids; mirror images and non-identical solids are separate components.
- The build stops rather than guess when a witness lies in zero or several solids, two solids share more than
  1e-3 mm³, a solid has no component, two components claim one solid, or a group lacks exactly its members.

## Feature witnesses

`features(params, shape)` gets a copy of `PARAMS` and the value `build` returned, after every cut, fillet and chamfer.
It returns JSON values only, keyed by the semantic ids the records use:

```json
{"hinge_bore": {"kind": "bore", "frame": "PART_LOCAL",
  "patches": [{"support": "CYLINDER", "point_mm": [12.0, 0.0, 4.0], "normal": [-1.0, 0.0, 0.0], "region": "left_cheek"}],
  "anchors": {"entry": {"point_mm": [9.0, 0.0, 8.0], "direction": [0.0, 0.0, -1.0]}},
  "relations": {"coaxial_with": ["press_band"]}}}
```

- `kind`, `region`, anchor and relation names are lower_snake_case; relations name ids of the same result.
- `support` is the kernel surface type: `PLANE`, `CYLINDER`, `CONE`, `SPHERE`, `TORUS`, `BEZIER`, `BSPLINE`,
  `REVOLUTION`, `EXTRUSION`, `OFFSET` or `OTHER`; `normal` and `direction` are unit vectors.
- One patch per surface span (both cheeks of a clevis bore). Each point lies on a final face found by a geometric
  selector, with that face's normal there.
- `frame` is the part's `model.frame` for `features` and `MANUFACTURE` for `manufacture_features`.
- A witness identifies a feature and carries no dimension: every measured value comes from the exported STEP.

## Manufacturing state

- `SAME`: the definition is the manufacturing state; the model defines no `build_manufacture`.
- `DIFFERENT`: `build_manufacture(params)` returns the state (a bent strap's flat blank) from the same `PARAMS`,
  sharing private helpers with `build`. A hook without the declaration, or the declaration without a hook, fails the
  build. PN, revision and material stay part.yaml's.
- Model the state in any native frame; the export pose comes from the route (references/exports.md "Export formats").
- The build fails when the state's solid count differs from the definition's, or its volume differs by more than
  part.yaml `manufacture.volume_tolerance`. Equal volume never proves the same holes, walls or material.
- A `manufacture_features` id equal to a `features` id declares a correspondence that `manufacture.decision` reviews;
  an installed feature without a state witness stays unmapped, never derived from the deformed shape.

## Robust geometry

- After each cut, fuse or intersect the result is valid with positive volume V, and V lies in: cut
  V(a) − ΣV(tools) ≤ V ≤ V(a); fuse max V(inputs) ≤ V ≤ ΣV(inputs); intersect 0 < V ≤ min V(inputs); with
  1e-7 relative (at least 1e-6 mm³) kernel noise. A violation raises and fails the build at that model line.
- A functional round (fit, seal, bearing, strength, a stated requirement) that fails stops the build. A cosmetic
  round may retry at r/2 and r/4 after OCCT `StdFail_NotDone`, then stay sharp; log each fallback with PN, feature
  and radius, and list it in the delivery `limits`.
- Keep outlines tangent-continuous under silhouette fillets: a fillet over a 2–5° kink left 0.03–0.14 mm edge
  tolerances and a STEP round-trip volume drift near 1e-5, which fails readback. Fix the model, never the bound.
- Put the kinks of two intersected outlines at different stations. Give T-intersecting bores different radii; equal
  radii left an unorientable face in CadQuery 2.8.
- Build axisymmetric form as one revolved profile, with fillets and chamfers drawn as profile arcs and lines.
