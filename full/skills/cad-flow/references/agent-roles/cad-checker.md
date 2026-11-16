# Checker

Owner: cad-flow. The `cad-checker` contract. It adds to the common contract [agent-roles](../agent-roles.md), which holds Every role, Read
set, Limits, Delivery fields, uv commands and Reply.

- Duty: independent checks of the brief's version or unit inputs: own programs, results, artifacts, logs, coverage; in a lite build also
  the LAYOUT intent inspection, a REVIEW result ([cad-verify families](../../../cad-verify/references/families.md#lite-intent-inspection))
- Skills a brief may list: cad-verify, cad-model, cad-select, cad-design, cad-release, cad-record
- Writes: `output/**`, `check.py`, `check.py.lock`, `tmp/**`, `__pycache__/**`, `delivery.json`; a lite geometry checker also `build.py`,
  `build.py.lock` when its brief lists them
- Delivers to: `hw/checks/<vN>/<check>/<attempt>/**`

## Checker programs

These extend [Every role](../agent-roles.md#every-role) step 6.

A checker whose `input_inventory` lists an earlier result's `source/check.py` and its lock may start its `check.py` from that copy
(cad-verify implementation kind, step 2): `cp <root>/.cad-work/<unit>/input/hw/checks/<vN>/<check>/<attempt>/source/check.py
<root>/.cad-work/<unit>/check.py`, and the same for `check.py.lock`. A lite geometry checker ([lead-side: stages Build
mode](../stages.md#build-mode)) also writes, locks and runs its own `build.py` before `check.py` when its brief's `boundary.writable`
lists `build.py` and `build.py.lock` ([cad-model exports — geometry.build](../../../cad-model/references/exports.md#geometrybuild), Lite),
and its `check.py` alone runs the brief's `freecad` tool, as a subprocess on the helper `tmp/<check>/freecad-readback.FCScript` it writes
([cad-model exports — FreeCAD readback](../../../cad-model/references/exports.md#freecad-readback)); never run `freecad` from the shell,
and never edit the helper after its run.
