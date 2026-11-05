# Build guide: `<product title>` REV `<REV>`

Template owner: cad-release (references/release.md `propose-release`). The lead fills every `<…>` field from the
frozen record of `<vF>` and the RELEASE verification index of `<vM>`, and deletes this template paragraph and the
next one. An UNKNOWN fact prints `UNKNOWN` with its reason; a physical test prints its outcome in the `<vM>` index
(`PASS`, `WAIVED` or `UNKNOWN`); nothing in this guide is stated beyond the record.

FULL build: delete the lite notice line below and the "Lite prototype" section. LITE build (references/release.md
"Lite release"): `<vF>` is the PASS LAYOUT version. Keep the lite notice line, which is verbatim the disclosure line
of cad-flow SKILL.md OUTPUT. Keep the Package, Scope and limits, What this package claims, Lite prototype, Not
established by this package and Licenses sections. Delete the full notice paragraph and every other section.

LITE prototype: geometry only; no material, fit, strength or manufacturing claims

This guide accompanies a reviewed model-generated definition package. Its drawings are not producer-ready and
claim conformance to no standard. Its geometry, simulation and review results are not physical validation.

## Package

| Field | Value |
|---|---|
| Product | `<id>`, `<title>` |
| Package revision | `<REV>` |
| Frozen version | `<vF>`, manifest sha256 `<sha256>` |
| Service class | `<service class>` |
| Release ceiling row | `<row of references/release.md "Release ceiling", or "LITE prototype" for a LITE build>` |
| Final manifest digest | reported with the release, outside the package |

## Scope and limits

- Purpose: `<release_scope.purpose>`
- Operating limits: `<each release_scope.operating_limits entry>`
- Excluded: `<each release_scope.exclusions entry>`
- Trial: `<waiver DR, envelope, supervision and expiry, or "none">`
- Waivers: `<each WV-### with its DR and limitation, or "none">`

## What this package claims

| Requirement | Facets | Outcome |
|---|---|---|
| `<REQ-###>` | `<facets>` | `<PASS, WAIVED or UNKNOWN, as the <vM> verification index states>` |

## Lite prototype

From the `<vF>` record snapshot and its accepted results.

- Build mode: `LITE`, a separate fact from the record plugin `<plugin name>` `<plugin version>`
- Parts: `<PN>` `<title>` at draft revision `<X revision>`; a draft revision is not a released part revision
- Dimensions: `<REQ-### and criterion id>` `<criterion with its bounds>`, measured `<value and unit>` by
  `<check>/<attempt>`
- Model sources and locks: `<each record/version/source/** path with its sha256>`
- Opening: open `definition/assembly.step` or `definition/parts/<PN>.step` (STEP AP214) in FreeCAD
  `<freecad_version>` with File > Open. `definition/assembly.glb` is visualization only, and `docs/previews/*.svg`
  are renders, never drawings
- FreeCAD check: FreeCAD `<freecad_version>`, OCCT `<occt_version>`: `<solid count>` solids, each valid and closed
  with its volume and bounding box within the check tolerance, by `<check>/<attempt>`. No other FreeCAD version
  was checked

## Parts

Copied from `bom.yaml`; `bom.csv` is its derived view.

| Path | PN | Rev | Title | Qty | Sourcing | Material profile | Route | Mass |
|---|---|---|---|---|---|---|---|---|
| `<path>` | `<PN>` | `<rev>` | `<title>` | `<quantity unit>` | `<sourcing>` | `<profile or UNKNOWN>` | `<route>` | `<mass or UNKNOWN>` |

## Making the parts

One subsection per made part, from its part record and `manufacturing/<PN>/`.

### `<PN>` `<title>`

- Files: `<manufacturing/<PN>/part.step, part.stl, flat.dxf or drawings/<PN>/sheet-N.pdf>`
- Route, in order: `<operations with their capability profile and approved orientation or setup>`
- Support, finish and secondary operations: `<as the record states>`
- Inspection: `<each drawing characteristic's inspection, or "none stated">`

## Purchased items

| PN | Identity | Specification | Supplier | Availability |
|---|---|---|---|---|
| `<PN>` | `<identity>` | `<specification>` | `<supplier or UNKNOWN>` | `<availability or UNKNOWN>` |

## Assembly

One step per `assembly.yaml` operation, in order.

1. `<operation id>`: `<description>`; tools `<tools>`; access `<access>`; retention `<retention>`; accept when
   `<acceptance criteria>`.

## Service

One entry per `assembly.yaml` service task: `<task id>`, `<description>`, operations `<operations>`, access
`<access>`.

## Not established by this package

- `<each UNKNOWN fact, UNKNOWN or WAIVED physical test and open question with its owner>`

## Licenses

- Hardware: `<license>`
- Documentation: `<license>`
- Third party: `<item, license, source, redistributed>`
