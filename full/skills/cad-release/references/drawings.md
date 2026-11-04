# Drawings

Owner: cad-release. A sheet here is a reviewed model-generated engineering drawing: a build123d
`TechnicalDrawing` program the model writes per version, or FreeCAD TechDraw when it is installed and its use is
approved. It is not producer-ready, certified, or conformant to ISO 128, ISO 129, ASME Y14.5 or any other
standard; a regime or general tolerance printed on a sheet repeats the record's statement and claims no
conformance. Intent fields are the part root's `drawing` block; paths are cad-record `layout.json`
(`drawing.sheet_svg`, `drawing.sheet_pdf`, `drawing.sheet_png`, `drawing.coverage`, `drawing.bundle_manifest`);
result fields are the `check-result` root; anchors and producers of the two drawing families are
[cad-verify Check families](../../cad-verify/references/families.md).

## Drawing intent

The part designer writes `drawing` in `hw/parts/<PN>/part.yaml` at DETAIL
([cad-design `detail-parts`](../../cad-design/references/parts.md#detail-parts)), from
`<skill-dir>/assets/drawing-intent.yaml`.

| Part | `drawing.status` |
|---|---|
| MADE_TO_DRAWING; or a characteristic needs a criterion the STEP cannot carry (fit, tolerance, thread, finish, edge break, inspection); or a requirement or the owner asks for a sheet | `REQUIRED` |
| BUY, identified by its designation or specification; or MAKE fully defined by its STEP, STL and route with no such characteristic | `NOT_REQUIRED`, with the reason |
| not yet decided | `UNKNOWN`: at DETAIL `drawing.coverage@<PN>` is a due UNKNOWN (`MISSING_INPUT`) |

- `datums`, `views` and `characteristics[]` (`id`, `feature`, `kind` DIMENSION, GEOMETRIC, FINISH, THREAD or
  NOTE, `criterion` with its source, `datums`, `inspection`, `function`) state what must be shown and accepted.
- A `feature` is a semantic feature id the model's `features` function witnesses
  ([cad-model Top-level names](../../cad-model/references/model-contract.md#top-level-names)), never a face, edge or
  solid index.
- A `criterion` comes from a requirement, interface, fit or the record's regime with its source; never from
  the model. The drawn value always comes from the STEP.
- The assembly has no drawing intent field in the record contract, so the `assembly` subject is never
  REQUIRED; assembly operations reach the build guide from `assembly.yaml` (references/release.md).

## draft-drawings

Runs in DETAIL and FREEZE after `build-version`, so every sheet binds the version manifest.

1. Subjects: every PN of the version whose snapshot `part.yaml` has `drawing.status: REQUIRED`. None →
   output "no REQUIRED drawing"; the drawing families' condition is false at `enumerate-obligations`.
2. Dispatch one `cad-part-designer` unit per subject in one batch (cad-flow references/delegation.md,
   references/worker-brief.md), skills including cad-release. `input_inventory`, copied into the unit's `input/`:
   `hw/versions/vN/manifest.json`, every file of `hw/versions/vN/record/**` and `hw/versions/vN/source/**`
   (the version programs and their locks), and the PN's STEP in the state the intent draws
   (`hw/versions/vN/definition/parts/<PN>.step` or `hw/versions/vN/manufacturing/<PN>/part.step`). Expected
   outputs: `sheet-<N>.svg`, `.pdf` and `.png` for every sheet, targets `hw/drawings/vN/<SUBJECT>/sheet-<N>.<ext>`.
3. The unit reproduces the feature witnesses, then writes, locks and runs `.cad-work/<unit>/build.py` (Drawing
   program); it reviews each PNG itself and reports every characteristic it could not draw as an UNKNOWN in the
   delivery.
4. Accept deliveries (cad-flow references/worker-delivery.md); publish the sheets byte for byte to
   `hw/drawings/vN/<SUBJECT>/`. A unit that delivers no sheet for a subject leaves `drawing.coverage@<PN>`
   UNKNOWN `MISSING_INPUT`.
5. `drawing.coverage` and `drawing.preview` run in `run-checks` (methods below). After coverage is accepted,
   write `hw/drawings/vN/<SUBJECT>/bundle-manifest.json`: `kind: DRAWING`, binding `VERSION`, `input` the
   manifest, the STEP, the record snapshot and the version programs read, `recipe` the drawing program's argv,
   packages and platform, `policy` from preflight, `files` every sheet (`DOCUMENT` for SVG and PDF, `IMAGE` for
   PNG), `coverage.json` (`DATA`) and the program `.cad-work/<unit>/build.py` and its lock (`SOURCE`, kept in
   scratch, no gate credit) → it validates.

## Drawing program

One build123d environment, pinned in the script header and locked as
[cad-model Kernels and environments](../../cad-model/references/model-contract.md#kernels-and-environments) states;
a CadQuery part's STEP is read with `import_step`, never by mixing CadQuery's OCP into this environment. Calls are
the ones [cad-model build123d calls](../../cad-model/references/build123d.md) checked.

- Feature locations: rerun the version programs from the input copy of `hw/versions/vN/source/` on the input copy
  of `hw/versions/vN/record/` into the unit's own scratch, CadQuery program first, as
  [cad-model Version build](../../cad-model/references/exports.md#version-build) step 4 runs them. The work
  directory then holds `signature.json` and `features/<PN>.json` (`features/<PN>.manufacture.json` for a
  manufacturing state) per [cad-model Signatures and witnesses](../../cad-model/references/exports.md#signatures-and-witnesses).
  A rebuilt signature that differs from the manifest's geometry stops the unit (MAJOR). Each characteristic's
  feature is found by its witness; a feature without a witness is UNKNOWN `MISSING_INPUT`.
- Views: project the STEP with `project_to_viewport(viewport_origin, viewport_up, look_at)`, which returns
  visible and hidden edges; place them on a `TechnicalDrawing(title, sub_title, drawing_number, sheet_number,
  drawing_scale, page_size, designed_by, design_date)` border. `drawing_number` is the PN; `sub_title` carries
  the part revision, `Units: mm`, the regime and the general tolerance as the record states them;
  `design_date` comes from the brief, never the clock.
- A characteristic whose feature no projected view shows gets a section (the part split by a plane, then
  projected) or a detail view. A characteristic that still cannot be shown is UNKNOWN `UNSUPPORTED_METHOD`.
- Dimensions: `ExtensionLine(border, offset, draft)` or `DimensionLine(path, draft)` on the projected edges
  or points nearest the named feature's witness, with `label` left unset so the printed value is the measured
  length, and `tolerance` taken from the criterion. Never type a numeric label.
- Fonts: `Draft(font=...)` takes a family name only (0.13.0 has no font path there) and `Text` takes
  `font_path`; give every `Text` the retained font file
  ([cad-model Text and fonts](../../cad-model/references/build123d.md#text-and-fonts)) and name that installed
  family in `Draft`, never the default `Arial`. Record family, file and license; a missing font is UNKNOWN
  `MISSING_INPUT`.
- Export with `ExportSVG(unit=Unit.MM)`: layers `Visible` and `Hidden`, every view, dimension and label added.
  PDF and PNG come from that SVG through an installed, approved converter or FreeCAD TechDraw, with its version
  recorded; PNG at 144 dpi or more. A required PDF or PNG without such a tool is UNKNOWN `UNSUPPORTED_METHOD`.
- Crowding never drops a dimension: add a sheet, a section or a detail view.
- No clock, network, random value or unsorted traversal; the same inputs give the same sheet text.

```text
visible, hidden = part.project_to_viewport((0, -200, 0), (0, 0, 1), look_at=(0, 0, 0))
d1 = ExtensionLine(border=bore_edge, offset=8 * MM, draft=draft, tolerance=criterion_tolerance)
```

## drawing.coverage

The method of family `drawing.coverage`: check `drawing-<subject in lowercase>`, attempt `a1`, obligation
`drawing.coverage@<PN>`, `facet: CAD_VALID`, result `hw/drawings/vN/<SUBJECT>/coverage.json`, binding
`VERSION`. A `cad-checker` unit runs it with its own program on the draft-drawings inputs: it reproduces the
witnesses the same way, reads the STEP and opens every actual SVG, PNG and PDF of the subject.

The coverage table is the result's `outcomes`, one entry per characteristic:

| Coverage column | `outcomes[]` field |
|---|---|
| feature | `subject {kind: FEATURE, PN, feature}` |
| measured value | `value`, from the STEP readback |
| criterion | `criterion`, from the intent |
| characteristic, sheet, view, printed annotation | `configuration`: `characteristic <id>; sheet <N>; view <name>; annotation '<printed text>'` |
| geometry and sheet hashes | `evidence`: the STEP, and each sheet file the annotation was read from |
| status | `outcome` |

- PASS: the annotation is printed on a sheet, legible in its PNG and present in its PDF; its value equals
  the measured value within the manifest's export tolerance; the printed tolerance equals the criterion.
- FAIL: a wrong value, a tolerance that is not printed, or an annotation on the wrong feature.
- UNKNOWN: the characteristic is not drawn (`UNSUPPORTED_METHOD`) or a sheet is missing (`MISSING_INPUT`).
- Every characteristic has one entry and every intent view exists, so `coverage_kind: FULL`. Feature
  recognition is not promised: a feature without a characteristic is no finding.

The result is PASS only when every entry is PASS.

## drawing.preview

The method of family `drawing.preview`: check id `drawing-preview`, obligations `drawing.preview@<PN>`,
`facet: CAD_VALID`, one result under `hw/checks/vN/drawing-preview/<attempt>/`, run by a `cad-checker` unit on
every sheet file actually present, with each file's sha256 in `artifacts`.

1. SVG: parses; page size equals the title-block page.
2. PDF: opens; one page per sheet with the same page size; its text includes every annotation of the SVG.
3. PNG: opens at the stated dpi; pixel size equals the page at that dpi; not blank; the checker views it as
   an image and confirms every view, dimension and the title block are visible and not clipped or overlapping.
4. A required format that is missing is UNKNOWN `MISSING_INPUT`; one the checker has no installed tool to
   open or render is UNKNOWN `UNSUPPORTED_METHOD`.

The result is PASS only when every outcome is PASS.

NOT Call a sheet producer-ready, or present the gate packet's previews as verification.
NOT Copy a value from a sheet, a draft or the model into part.yaml.
