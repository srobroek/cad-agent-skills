---
name: cad-model
description: Writes one-kernel part models, then builds, exports and reads back their geometry. Use for "write the model.py", "build the version", "export STEP", "read back".
---

# CAD Model

TRIGGER
+ "write the model.py", "add a bore to the base", "build the parts", "export STEP", "export the STL for printing"
+ "read back the geometry", "is the B-rep valid", "why did the fillet fail", "build in build123d"
+ cad-flow `build-version` for a stage whose policy lists `geometry.build`
+ a brief that builds geometry, or a checker brief naming a `geometry.*` or `preview.render` check
- what a part, interface or assembly should be → cad-design
- clearance, motion, fit, process, load or mass checks → cad-verify
- drawings, BOM or release package → cad-release; stage order or approval → cad-flow

## Workflow

1. Read the part records the brief or stage names: `hw/parts/<PN>/part.yaml` (model kind, kernel, frame, components,
   manufacture, route), `hw/assembly.yaml` and `hw/interfaces.yaml` → the PNs, kernels and features to build.
2. LOAD references/model-contract.md (`skill://cad-model/references/model-contract.md`); write or change each
   `model.py` → top-level `KERNEL`, `PARAMS` and `build`, plus the declared hooks.
3. LOAD references/cadquery.md (`skill://cad-model/references/cadquery.md`) or references/build123d.md
   (`skill://cad-model/references/build123d.md`) for the model's kernel → only the calls it verifies, at its pins.
4. LOAD references/exports.md (`skill://cad-model/references/exports.md`); write the build program per "Unit build"
   or "Version build", lock it, and run it locked → exit 0 and every export at its layout path.
5. Read every export back with a separate build123d program per "Readback" → the expected check results, PASS or
   the FAIL or UNKNOWN they found, each with its numbers.
6. Report the result files, logs and every cosmetic fallback; a worker fills its delivery per its role reference.

## Rules

MUST Use one kernel per `model.py` and per process; geometry crosses the CadQuery and build123d environments only as
  STEP files.
MUST Keep `PARAMS` the single authority for dimensions, in mm and degrees; records hold criteria and sources only.
MUST Keep models and programs deterministic: no clock, network, randomness, unsorted traversal, host font or
  read of a mutable record file; text only from a retained, identified font file.
MUST Run every build and check as a locked script (`uv lock --script`, then `uv run --locked --script`).
MUST Stop on a failed build, Boolean bound or functional round; report a cosmetic fallback, never hide it.
NOT Name geometry by face, edge or solid index, or guess a frame or orientation from the shape.
NOT Treat a successful build, a preview or an exit code as readback, verification or manufacturability.
NOT Ship or run a model, runner or helper from this skill; every program belongs to the product project.

OUTPUT
L1 `MODEL <unit-or-vN>: BUILT|FAILED — <n> parts, <kernels> — readback PASS|FAIL|UNKNOWN`
   Failures — PN, model line, check and value — only if any
   Fallbacks — PN, feature and radius of each skipped cosmetic round — only if any
   Files — one absolute path per line
CAP 100w clean · 220w with failures
