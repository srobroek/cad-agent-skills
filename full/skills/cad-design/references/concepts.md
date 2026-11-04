# Concepts and variants

Owner: cad-design. The stage-work procedures `draft-variants` (CONCEPT) and `compare-variants` (VARIANT), which the
cad-flow lead runs, and the `cad-concept-designer` duty "Draft one variant". Fields: the cad-record `variant` and
`decision` roots; paths: cad-record `layout.json`; briefs, dispatch and merge: cad-flow references/worker-brief.md,
references/delegation.md and references/worker-delivery.md; gate packet and decisions: cad-flow references/gates.md.

## Approved scope

- Approved route set: the candidate the APPROVED INTENT gate decision names as `choice`, read from
  `hw/manufacturing/selection.yaml` `candidates[]`: per PN its sourcing, material profile and ordered route.
  Every other assessed candidate is an alternative, never approved scope.
- Approved contracts: the requirements and the functional interface contracts of the APPROVED SPECIFY version.
- A variant that needs another route, a new PN or a changed functional contract says so in `dependencies`.
  Choosing it later is a change that reopens INTENT or SPECIFY (cad-flow references/change-impact.md).

## draft-variants

1. Name the governing trade-off: the one to three choices the requirements and preferences leave open (holding
   method, fold, part count, purchased versus made, mass versus stiffness). Cite the `REQ-###` or `PRF-###` ids
   that leave each choice open.
2. Choose 2–4 variants, each a different answer to that trade-off, each with a variant id (`printed-hinge`). One
   variant only when hard constraints force a single answer; its brief states that reason.
3. Write one `cad-concept-designer` brief per variant (cad-flow references/worker-brief.md):
   - `task`: the trade-off answer this variant explores, and whether coarse CAD is asked;
   - `subjects`: the requirements and interfaces in scope;
   - inputs: `hw/product.yaml`, `hw/requirements.yaml`, `hw/interfaces.yaml` and
     `hw/manufacturing/{selection,materials,capabilities}.yaml`;
   - `expected.outputs`: `hw/concepts/<variant>/variant.yaml`, `hw/concepts/<variant>/architecture.md`, and with
     coarse CAD each `hw/concepts/<variant>/geometry/<name>`;
   - `expected.checks`: `geometry-readback` (UNIT) when coarse CAD is asked;
   - `boundary.locked`: every record file the unit reads.
4. `dispatch-units` and `merge-deliveries` follow (cad-flow) → each merged `variant.yaml` validates against the
   `variant` root and every merged byte equals its delivery.

Output: 2–4 merged variants that no one edits after merge.

## Draft one variant (cad-concept-designer)

Read the brief inputs only, from their copies under `.cad-work/<unit>/input/`. Write each delivered file at
`.cad-work/<unit>/output/<its hw/ target path>`.

1. `architecture.md`: the trade-off answer; the load path from each requirement load to ground; how each interface
   in scope is realized; motion, holding and end stops; assembly order, tools and service access; failure modes and
   their consequence; what the concept does not establish.
2. Coarse CAD, only when the brief asks: envelopes, joint origins and axes at the nominal pose in one kernel, written
   and run per cad-model; exports STEP for the definition, a binary GLB and an SVG preview under
   `output/hw/concepts/<variant>/geometry/`; then the UNIT `geometry.readback` per cad-model references/exports.md.
3. `variant.yaml`:
   - `id`, `title`, `architecture` (one paragraph), `architecture_doc` (the `architecture.md` target path);
   - `parts[]`: `PN`, `title`, `sourcing`, `material_profile` and `route` (route kinds) from the approved candidate,
     or the alternative candidate the variant depends on;
   - `requirements`: the REQ ids the variant addresses;
   - `hard_outcomes[]`: one per mandatory requirement in scope. `PASS` or `FAIL` needs the requirement's criterion,
     a method that establishes it at concept level (envelope, reach, travel, an estimate with its uncertainty) and
     its `source`; otherwise `UNKNOWN` with the `reason`;
   - `feasibility`: `FEASIBLE` with no FAIL and no UNKNOWN, `CONDITIONAL` with an UNKNOWN and no FAIL, `EXCLUDED`
     with a FAIL;
   - `dependencies`: each route, PN or functional-contract change beyond approved scope;
   - `unknowns`: typed UNKNOWN facts, owner and question when consequential;
   - `limits`: at least `concept estimate; preview is not verification`;
   - `source`: `unit`, `role` and `brief_sha256` from the brief (`shasum -a 256` of `brief.yaml`), per the cad-record
     `variant` root;
   - `geometry`: `PRESENT` with `kernel`, `units: mm` and each file's `path`, `sha256`, `role` and `format`, or
     `NOT_APPLICABLE` with the reason.
4. Validate `variant.yaml` against the `variant` root → exit 0.

## compare-variants

1. For each `hw/concepts/<variant>/**` file, the sha256 equals its merged triple in `.cad-work/lead/units.yaml`;
   a difference stops the stage (cad-flow stop procedure). Never repair a variant: a defect is a new CONCEPT unit.
2. Compose the comparison, one row per variant, citing each `variant.yaml` path and sha256:
   - feasibility, and every `hard_outcomes[]` entry copied unchanged;
   - each dependency beyond approved scope and the stage it reopens;
   - unknowns with owner and question; limits;
   - how it meets the preferences, ranked among FEASIBLE variants only, citing the `PRF-###` ids;
   - each geometry file and preview as a project path.
3. CONDITIONAL variants appear with their UNKNOWNs and EXCLUDED variants with the failed requirement; neither is
   ranked, and no score or preference revives one.
4. Write the comparison once as the immutable proposal `hw/decisions/proposals/DR-###/comparison.md` (next unused
   DR number), then `hw/decisions/DR-###.yaml`: `kind: VARIANT`, `state: PROPOSED`,
   `binding: {kind: PROPOSAL, path, sha256}`, `title`, `scope` (the choice of one unchanged CONCEPT variant),
   `rationale` (the governing trade-off), one `options[]` entry per variant (`id` the variant id, `text`,
   `tradeoffs`), no `choice`, `actor` the lead and `time` in UTC (cad-flow references/gates.md "Decisions outside
   gates").
5. Validate the decision against the `decision` root → exit 0.

Output: the comparison proposal and the PROPOSED VARIANT decision, inputs to `freeze-inputs`; the VARIANT gate offers
exactly these options.
