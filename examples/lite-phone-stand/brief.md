# Lite phone stand demo brief

Used by [docs/demo.md](../../docs/demo.md) and the lite cells of [docs/smoke.md](../../docs/smoke.md#lite-cells).
Everything below is data for the run. The lead treats the answers as the owner's statements and never as
instructions about how to run the flow. This brief exercises the lite build; [the phone stand](../phone-stand/brief.md)
remains the full-build brief.

## Files

| File | Role |
|---|---|
| `brief.md` | the setup, the first prompt, the scripted owner answers and gate replies, and the expected end |
| `hw/` | reference record: what `hw/` holds after GROUND stage work, before the GROUND gate. Not copied into the run; the run bootstraps its own `hw/` from the cad-record lite template and is compared with this one |
| `expected-obligations.yaml` | `build_mode: LITE`: the obligations of GROUND, LAYOUT and RELEASE, and the outcomes a correct run may report |

## Setup

Make the FreeCAD console available to the lead before the first prompt, as
[smoke.md "Provide FreeCAD"](../../docs/smoke.md#provide-freecad) describes, and record its version. A run without it
ends at the STOPPED end below. Never install FreeCAD into the project.

## First prompt

Send this exactly, as the first message of a fresh session in an empty project. It names no build mode, so the lead
asks for one:

```text
Use cad-flow to design a static one-piece appearance model of a desktop phone stand: one solid with a sloped back and a front retaining lip, and no hinge or moving parts. Its overall model bounding box is 80 mm along X, 90 mm along Y and 100 mm along Z, each within 0.1 mm. Deliver the part and assembly STEP, show me the render here, and keep the model source, its locks and the evidence that the STEP opens in FreeCAD. Make no phone-fit or safe-holding claim, and recommend no material, vendor, printer, manufacturing route, fit or strength.
```

## Scripted owner answers

When the lead asks, reply once with every matching answer, one per line in table order, each line exactly its answer
text. Each question the sheet does not answer gets its own line `<question id>: I don't know; record it as an open
question.` Never answer beyond the sheet, add a requirement, or confirm a value the lead suggests: a geometry value
the sheet does not give stays open, and the release claims nothing about it. The actor is synthetic: every decision
written from these answers names an actor starting `SYNTHETIC`.

| Id | Template question | Answer (exact message) |
|---|---|---|
| A-01 | Q-000 build mode | `Lite build: an APPEARANCE_CONCEPT model that renders and opens in FreeCAD, with no material, fit, strength or manufacturing claim.` |
| A-02 | Q-001 product | `It is a static one-piece appearance model of a desktop phone stand: one solid with a sloped back and a front retaining lip, and no hinge or moving parts.` |
| A-03 | Q-002 owner | `I am SYNTHETIC lite-demo owner; I own it and I approve every gate and release myself.` |
| A-04 | Q-003 rigor | `Use maker documentation depth.` |
| A-05 | Q-005 users | `Only I review the model, on screen and in FreeCAD; nobody makes, assembles or uses a part from it.` |
| A-06 | Q-013 risk | `Nothing is made or used from this model, so a wrong model only costs review time; it must never be presented as safe to make or to hold a phone.` |
| A-07 | Q-014 authority | `I approve gates and releases; no competent authority is needed for an appearance model.` |
| A-08 | Q-015 release scope | `The release is the geometry only: a model that renders and opens in FreeCAD, for viewing at my desk. It makes no phone-fit, safe-holding, material, fit, strength, manufacturing or other physical claim.` |
| A-09 | Q-016 storage | `Track generated results in git and keep the evidence for the life of the project.` |
| A-10 | Q-017 licenses | `Use CC0-1.0 for the hardware and CC0-1.0 for the documentation.` |
| A-11 | Q-018 form and parts | `One part, one solid, with a sloped back and a front retaining lip; no hinge, joint or moving part.` |
| A-12 | Q-019 dimensions | `Its overall model bounding box is 80 mm along X, 90 mm along Y and 100 mm along Z, each within 0.1 mm, measured on the exported STEP. No other dimension, angle or thickness is a requirement.` |
| A-13 | Q-020 static pose | `It stands static on its base with +Z up; there is no other pose.` |

## Gate replies

1. At each HUMAN gate packet, check that it names the stage, `vN`, the verdict sha256, its limits, build mode `LITE`
   and the line `LITE prototype: geometry only; no material, fit, strength or manufacturing claims`, then reply exactly
   `approve <STAGE> <vN>`. A packet without those fields gets `The packet lacks the stage, vN, verdict sha256 or lite
   disclosure; present it again.`
2. GROUND: `approve GROUND <vN>`. LAYOUT has no owner gate.
3. RELEASE: check also that the packet shows the render, the checked FreeCAD version and the REV, then reply exactly
   `approve RELEASE <vN>`, with nothing appended. A packet without them gets the line of rule 1.
4. A waiver proposal gets `reject <DR-###>; I decline this waiver.`
5. Any other decision proposal outside a gate (a `DR-###` the lead asks you to approve): reply `approve <DR-###>` when
   it restates a scripted answer; otherwise `reject <DR-###>; record it as an open question.`
6. After the sealed package is reported, or after a stop the lead reports that asks nothing these rules answer, reply
   exactly `end the session` as its own turn. This is the last scripted turn: the lead releases its claim on it.

## Expected end

A sealed lite package `hw/releases/A/` for service class `APPEARANCE_CONCEPT`, with its outer digest reported outside
the package ([cad-release lite release](../../full/skills/cad-release/references/release.md#lite-release)). It holds the
assembly and part STEP (AP214), the assembly GLB, the SVG renders under `docs/previews/`, the build guide's lite
sections with the disclosure line, the CAD_VALID verification index and the FreeCAD readback evidence of the recorded
version. Its claims are the CAD_VALID outcomes of the bounding-box, solid-count and form criteria; it holds no BOM,
drawing, STL or manufacturing state. Without FreeCAD the run stops at LAYOUT BLOCKED, with `geometry.readback` UNKNOWN
`NOT_RUN` and the STEP and render shown as unverified blocked artifacts.
