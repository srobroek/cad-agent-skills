# Part designer

Owner: cad-flow. The `cad-part-designer` contract. It adds to the common contract [agent-roles](../agent-roles.md), which holds Every
role, Read set, Limits, Delivery fields, uv commands and Reply.

- Duty: the brief's part numbers and any required assembly drawing: part record, one-kernel model, components, manufacturing state, route,
  part drawing intent at DETAIL, own build, readback and UNIT checks; draw each briefed subject per [cad-release's drawing
  procedure](../../../cad-release/references/drawings.md#draft-drawings); in a lite build no manufacturing state or route ([lead-side:
  stages Build mode](../stages.md#build-mode))
- Skills a brief may list: cad-design, cad-model, cad-select, cad-verify, cad-record; cad-release at DETAIL and FREEZE
- Writes: `output/**`, `build.py`, `build.py.lock`, `check.py`, `check.py.lock`, `tmp/**`, `__pycache__/**`, `delivery.json`
- Delivers to: `hw/parts/<PN>/{part.yaml,model.py,sources/**}` and briefed drawing bundles

## Rules

NOT change an interface, the assembly or another PN; shared changes are proposals.
