# Layout and interfaces

Owner: cad-design. The stage-work procedures `specify-interfaces` (SPECIFY) and `define-layout` (LAYOUT), which the
cad-flow lead runs. Fields: the cad-record `interfaces`, `assembly` and `part` roots; paths: cad-record `layout.json`;
what an edit reopens: cad-flow references/change-impact.md. Placement and frame rules: cad-model
references/model-contract.md "Frames and placement".

## specify-interfaces

Input: this attempt's requirements (cad-verify `specify-requirements`), the approved selection candidate (the `choice`
of the APPROVED INTENT gate decision in `hw/manufacturing/selection.yaml`) and `hw/product.yaml`.
Output: `hw/interfaces.yaml` (root `interfaces`), written by the lead.

1. List every functional boundary the requirements and the approved candidate's architecture imply: part to part,
   part to purchased item, and product to user, environment, tool or service. One interface each, `IF-###` with the
   next unused number; an id is never reused.
2. `members[]`: `PN` from the approved candidate, `feature` a semantic id naming the functional feature
   (`hinge_bore`, `phone_seat`), `role`. A boundary with the user, the environment or a tool lists the product
   member only; `environment` and `access` describe the counterpart.
3. `title`, `kind`, `functions[]`, `requirements` (the REQ ids it serves), and `critical`: KNOWN `true` when a
   requirement's acceptance, a load path, motion, sealing, retention or safety depends on the interface.
4. Every other required field is KNOWN with its source or a typed UNKNOWN, with owner and question when
   consequential:
   - `fit`: kind, designation from a retained standard, `state` and `source`; or open until DETAIL;
   - `clearance`: the minimum and its criterion from a requirement;
   - `play`: axial, radial and angular;
   - `variation`: the stack case and `WORST_CASE`, or `JUSTIFIED_RSS` with its approving decision;
   - `contact`: materials from the approved profiles, finish, lubrication, load, speed, duty and life;
   - `environment`; `access`: tool, install, service and inspection.
5. A critical interface lists `datums` (`PRIMARY`, `SECONDARY`, `TERTIARY` on member features) and `acceptance`
   criteria copied from its requirements.
6. Never fill a clearance, fit class, tolerance, lubrication, load or life without its source. An ISO fit designation
   says nothing about as-printed precision (cad-verify references/fits.md).
7. Validate against the `interfaces` root → exit 0; list each consequential UNKNOWN as an open question for the
   SPECIFY gate.

Done when every requirement about an interaction cites an interface, and no interface has an absent or defaulted
field (`interfaces.completeness`).

## define-layout

Input: the variant the APPROVED VARIANT gate decision names as `choice` (`hw/concepts/<variant>/**`, unchanged), the
approved requirements, interfaces and selection.
Output: `hw/assembly.yaml` (root `assembly`), member placement in `hw/interfaces.yaml`, and the part units below.

1. PNs: keep every PN the chosen variant and the approved candidate use. A new part takes the next unused number
   with the product's PN prefix (`[A-Z]{2,5}-[0-9]{4}`) and revision `X1`; a PN is never reused.
2. `occurrences[]`: one per physical instance, `id` a semantic id, `PN`, `rev`, `parent` (an occurrence id; absent
   for a top-level occurrence) and `placement`: `translation` in mm and `rotation` a unit quaternion `[w, x, y, z]`
   in the parent frame, applied rotation first, then translation. `root` names the top occurrence when one exists.
3. `joints[]`: one per relative motion a requirement needs: `kind`, `parent` and `child` occurrences, `origin` and
   `axis` in the parent frame, `limits` over the full stated travel of the requirement (`mm` or `deg`), `zero` the
   nominal pose, `interface` the IF it realizes, and `source`. A rigid attachment (captured nut, bonded foot,
   fastened bracket) is a placement under its parent or a `FIXED` joint, never a moving joint that demands a bore.
   A joint that realizes the motion of a MOTION case in `hw/check-plan.yaml` takes that case's joint id
   ([cad-verify plan-checks](../../cad-verify/references/check-plan.md#plan-checks) step 3); a differing joint is a check-plan
   change.
4. `couplings[]`: joints moved together by a gear, linkage, cam, belt or screw, with `relation` and `ratio`.
5. `operations[]`: the one assembly-operations authority. Each install, fasten, press-fit, heat-set, retaining-ring,
   join, close, adjust, inspect and disassemble step names `kind`, `occurrences`, `into`, `interfaces`,
   `predecessors`, `direction` (a vector in the assembly frame), `tool_access` (each tool with its approach, reach
   and envelope) and `acceptance`; `retention`, `service_access`, `engagement` and `torque` are KNOWN with their
   source or typed UNKNOWN. A part installed before a closure is ordered before the `CLOSE` step. `service[]` tasks
   name their operations and access.
6. Interface placement: set each member's `occurrence`; place each datum on a placed member feature. A functional
   contract the variant needs that SPECIFY did not approve stops the stage: a CHANGE proposal reopens SPECIFY.
7. Part units, one `cad-part-designer` unit per group of PNs that share one kernel and interact (cad-flow
   references/worker-brief.md):
   - `task`: the LAYOUT model of each PN per references/parts.md "Fidelity by stage", in the kernel the task names
     (an existing part keeps its `model.kernel`);
   - `subjects`: the PNs; inputs: assembly, interfaces, requirements, selection, the chosen variant;
   - `expected.outputs`: `hw/parts/<PN>/part.yaml` and, for an AUTHORED or PROXY model, `hw/parts/<PN>/model.py`;
   - `expected.checks`: `geometry-build`, `geometry-readback` and `geometry-identity` (UNIT);
   - `boundary.locked`: interfaces, assembly, requirements, selection and every other PN.
8. A BUY part needs its supplier identity or model: a `cad-sourcing-researcher` unit for the PN's purchased family
   (cad-select), whose retained model or facts the part unit then cites.
9. Validate `assembly.yaml` and `interfaces.yaml` → exit 0. Occurrence-tree uniqueness and acyclicity, and unit
   quaternions, are cad-record semantic checks.

Done when every occurrence has a PN with a part unit, every joint limit comes from a requirement or is a typed
UNKNOWN, every interface member is placed, and every operation states its retention and access.
