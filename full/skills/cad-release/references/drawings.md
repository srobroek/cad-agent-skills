# Drawings

Owner: cad-release. A sheet here is a reviewed model-generated engineering drawing: a build123d
`TechnicalDrawing` program the model writes per version, or FreeCAD TechDraw when it is installed and its use is
approved. It is not producer-ready, certified, or conformant to ISO 128, ISO 129, ASME Y14.5 or any other
standard; a regime or general tolerance printed on a sheet repeats the record's statement and claims no
conformance. Intent fields are the part root's `drawing` block; paths are cad-record `layout.json`
(`drawing.sheet_svg`, `drawing.sheet_pdf`, `drawing.sheet_png`, `drawing.coverage`, `drawing.source`,
`drawing.artifacts`, `drawing.bundle_manifest`);
result fields are the `check-result` root; anchors and producers of the two drawing families are
[cad-verify Check families](../../cad-verify/references/families.md).

## Drawing intent

The part designer writes `drawing` in `hw/parts/<PN>/part.yaml` at DETAIL
([cad-design `detail-parts`](../../cad-design/references/parts.md#detail-parts)), from
`<skill-dir>/assets/drawing-intent.yaml`. The lead writes the same block as `drawing` in `hw/assembly.yaml` (the
assembly root's optional drawing intent) when the assembly needs a sheet, for example to show insertion order,
joint ranges or service access.

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
- An assembly characteristic names a feature id that exactly one occurrence's part witnesses; the unit places it
  through that occurrence; an id two occurrences witness is UNKNOWN `MISSING_INPUT`. Operations stay in
  `assembly.yaml` `operations` and reach the build guide from there (references/release.md).

## draft-drawings

Runs in DETAIL and FREEZE after `build-version`, so every sheet binds the version manifest.

1. Subjects: every PN of the version whose snapshot `part.yaml` has `drawing.status: REQUIRED`, and `assembly` when
   the snapshot `assembly.yaml` has `drawing.status: REQUIRED`. None → output "no REQUIRED drawing"; the drawing
   families' condition is false at `enumerate-obligations`.
2. Dispatch one `cad-part-designer` unit per subject in one batch (cad-flow references/delegation.md,
   references/worker-brief.md), skills including cad-release. `input_inventory`, copied into the unit's `input/`:
   `hw/versions/vN/manifest.json`, every file of `hw/versions/vN/record/**` and `hw/versions/vN/source/**`
   (the version programs and their locks), and the STEP in the state the intent draws
   (`hw/versions/vN/definition/parts/<PN>.step` or `hw/versions/vN/manufacturing/<PN>/part.step`;
   `hw/versions/vN/definition/assembly.step` for `assembly`). Expected outputs: `sheet-<N>.svg`, `.pdf` and `.png`
   for every sheet, targets `hw/drawings/vN/<SUBJECT>/sheet-<N>.<ext>`; `build.py` and `build.py.lock`, targets
   `hw/drawings/vN/<SUBJECT>/source/`; `font-check.json`, target `hw/drawings/vN/<SUBJECT>/artifacts/font-check.json`.
3. The unit reproduces the feature witnesses, runs the font check (Drawing program), then writes, locks and runs
   `.cad-work/<unit>/build.py`; it reviews each PNG itself and reports every characteristic it could not draw as an
   UNKNOWN in the delivery. A font check that does not match delivers MAJOR with `ready: false`; the lead stops
   the stage BLOCKED (`DELIVERY_FAILED`) with the font named, and no sheet is published.
4. Accept deliveries (cad-flow references/worker-delivery.md); publish every accepted output byte for byte to its
   target. A unit that delivers no sheet for a subject leaves `drawing.coverage@<PN>` UNKNOWN `MISSING_INPUT`.
5. `drawing.coverage` and `drawing.preview` run in `run-checks` (methods below). After coverage is accepted,
   write `hw/drawings/vN/<SUBJECT>/bundle-manifest.json`: `kind: DRAWING`, binding `VERSION`, `input` the
   manifest, the STEP, the record snapshot and the version programs read, `recipe` the drawing program's argv,
   packages and platform, `policy` from preflight, `files` every sheet (`DOCUMENT` for SVG and PDF, `IMAGE` for
   PNG), `coverage.json` (`DATA`), `artifacts/font-check.json` (`ARTIFACT`), and the drawing and coverage programs
   with their locks under `source/` (`SOURCE`, `LOCK`), each by its `hw/drawings/vN/<SUBJECT>/` path → it validates.

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
- Views: project the STEP orthographically with `project_to_viewport(viewport_origin, viewport_up, look_at)` (no
  `focus`), which returns visible and hidden edges, and place them at 1:1 on a `TechnicalDrawing(title, sub_title,
  drawing_number, sheet_number, drawing_scale, page_size, designed_by, design_date)` border with `drawing_scale` 1.
  `Draft` has no scale and prints the 2D length of the path it is given, so a scaled view prints scaled values: fit
  a large part with a larger `page_size`, another sheet or a partial view, never by scaling. `drawing_number` is
  the PN; `sub_title` carries the part revision, `Units: mm`, the regime and the general tolerance as the record
  states them; `design_date` comes from the brief, never the clock.
- A characteristic whose feature no projected view shows gets a section (the part split by a plane, then
  projected) or a detail view. A characteristic that still cannot be shown is UNKNOWN `UNSUPPORTED_METHOD`.
- Dimensions: `ExtensionLine(border, offset, draft)` or `DimensionLine(path, draft)` on the projected edges
  or points nearest the named feature's witness, only where that edge or point pair lies parallel to the view
  plane, so its projected length is its true length; otherwise use a section or a view normal to it. `label`
  stays unset so the printed value is the measured length, `tolerance` comes from the criterion, and
  `Draft(decimal_precision=…)` prints every limit of that tolerance without rounding. Never type a numeric label.
- Fonts: the font file and its license come from the version record, `hw/versions/vN/record/parts/<PN>/sources/`
  (for `assembly`, the part sources the brief names)
  ([cad-model Determinism](../../cad-model/references/model-contract.md#determinism),
  [Text and fonts](../../cad-model/references/build123d.md#text-and-fonts)). `Text` takes it by `font_path`.
  `Draft(font=...)` takes a family name only (0.13.0 has no font path there), so dimension labels use the installed
  family of that same file; this family name is allowed only after the font check below passes, never `Arial`.
  A missing font file is UNKNOWN `MISSING_INPUT`.
- Font check: `ExportSVG` writes every label as glyph outlines, so a sheet names no font. Before drawing, render
  the calibration string `0123456789.,+-±°ØRM` at the sheet's `font_size` once with `Text(..., font_path=<retained
  file>)` and once with `Text(..., font=<Draft family>)`; their face area and bounding-box size must be equal
  within 1e-6 mm² and 1e-6 mm. Write both values, the family, the file path and its sha256 to
  `.cad-work/<unit>/output/font-check.json` (role `DATA` in the delivery; published as
  `hw/drawings/vN/<SUBJECT>/artifacts/font-check.json`, role `ARTIFACT` in the drawing bundle). Unequal values
  mean the family resolves to another font: stop, never draw.
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
`drawing.coverage@<PN>` or `@assembly`, `facet: CAD_VALID`, result `hw/drawings/vN/<SUBJECT>/coverage.json`, binding
`VERSION`. A `cad-checker` unit runs it with its own program on the draft-drawings inputs: it reproduces the
witnesses the same way, reads the STEP and opens every actual SVG, PNG and PDF of the subject. The lead retains
that program and its lock under `hw/drawings/vN/<SUBJECT>/source/`, and `coverage.json`
`implementation.source_files` name those published paths with their sha256; neither `source_files` nor
`artifacts` lists `coverage.json` itself.

The coverage table is the result's `outcomes`, one entry per characteristic:

| Coverage column | `outcomes[]` field |
|---|---|
| feature | `subject {kind: FEATURE, PN, feature}` |
| measured value | `value`, from the STEP readback |
| criterion | `criterion`, from the intent |
| characteristic, sheet, view, printed annotation | `configuration`: `characteristic <id>; sheet <N>; view <name>; annotation '<printed text>'` |
| geometry and sheet hashes | `evidence`: the STEP, and each sheet file the annotation was read from |
| status | `outcome` |

- PASS: the annotation is printed on a sheet, legible in its PNG and on its rendered PDF page (labels are glyph
  outlines, read as images); its printed value equals the STEP-measured value rounded to its printed decimal
  places, and those places print every limit of the criterion's tolerance without rounding; the printed tolerance
  equals the criterion.
- FAIL: a wrong value, too few decimal places for the tolerance, a tolerance that is not printed, or an annotation
  on the wrong feature.
- UNKNOWN: the characteristic is not drawn (`UNSUPPORTED_METHOD`) or a sheet is missing (`MISSING_INPUT`).
- Every characteristic has one entry and every intent view exists, so `coverage_kind: FULL`. Feature
  recognition is not promised: a feature without a characteristic is no finding.

The result is PASS only when every entry is PASS.

## drawing.preview

The method of family `drawing.preview`: check id `drawing-preview`, obligations `drawing.preview@<PN>` and
`drawing.preview@assembly` per REQUIRED subject, `facet: CAD_VALID`, one result under
`hw/checks/vN/drawing-preview/<attempt>/`, run by a `cad-checker` unit on every sheet file actually present, with
each file's sha256 in `artifacts`.

1. SVG: parses; page size equals the title-block page.
2. PDF: opens; one page per sheet with the same page size; each page rendered at the PNG's dpi shows the same
   views, dimensions and title block as the PNG. Labels are outlines, so the PDF has no text layer to compare.
3. PNG: opens at the stated dpi; pixel size equals the page at that dpi; not blank; the checker views it as
   an image and confirms every view, dimension and the title block are visible and not clipped or overlapping.
4. Font: the producer's `font-check.json` names a font file whose sha256 equals its copy under
   `hw/versions/vN/record/parts/<PN>/sources/`, with equal values; the checker repeats the font check in its own
   environment and gets equal values. A missing record, a different file or unequal values is UNKNOWN
   `MISSING_INPUT` (the required font is not the one in use), so the stage is BLOCKED.
5. A required format that is missing is UNKNOWN `MISSING_INPUT`; one the checker has no installed tool to
   open or render is UNKNOWN `UNSUPPORTED_METHOD`.

The result is PASS only when every outcome is PASS.

NOT Call a sheet producer-ready, or present the gate packet's previews as verification.
NOT Copy a value from a sheet, a draft or the model into part.yaml.
