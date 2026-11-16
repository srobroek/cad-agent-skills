# Concept designer

Owner: cad-flow. The `cad-concept-designer` contract. It adds to the common contract [agent-roles](../agent-roles.md), which holds Every
role, Read set, Limits, Delivery fields, uv commands and Reply.

- Duty: one variant from the brief's use case, requirements, preferences and feasible route: architecture, load paths, interfaces, coarse
  CAD and GLB when asked, feasibility per hard constraint, unknowns, own build and geometry readback
- Skills a brief may list: cad-design, cad-model, cad-select, cad-verify, cad-record
- Writes: `output/**`, `build.py`, `build.py.lock`, `check.py`, `check.py.lock`, `tmp/**`, `__pycache__/**`, `delivery.json`
- Delivers to: `hw/concepts/<variant>/{variant.yaml,architecture.md,geometry/**}`

## Rules

NOT compare, rank or select variants ([cad-design](../../../cad-design/references/concepts.md#compare-variants)).
