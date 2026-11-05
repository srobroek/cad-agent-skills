# Part design and detail

Owner: cad-design. The stage-work procedures `design-parts` (PARTS) and `detail-parts` (DETAIL), which the cad-flow
lead runs, and the `cad-part-designer` duty "Design one part". Fields: the cad-record `part` root; paths: cad-record
`layout.json`; model program, frames, components and manufacturing state: cad-model references/model-contract.md;
fit limits and process criteria: cad-verify; drawing production: cad-release references/drawings.md.

## Fidelity by stage

Fidelity describes the form a stage expects; it is review language, never a check result or a pass condition.

| Stage | Made part | Purchased part |
|---|---|---|
| LAYOUT | envelope solids at the `assembly.yaml` placement; joint origins and axes and every interface member feature witnessed | supplier model, or a PROXY with sourced bounds |
| LAYOUT, LITE build | the finished prototype form the approved GROUND requests: every opening, wall, bore, round, lip and named feature at its requested dimension, at the `assembly.yaml` placement; never an envelope stand-in | none: a lite part is AUTHORED ([Lite layout](layout.md#lite-layout)) |
| PARTS | real interfaces and route-appropriate form: bores, bosses, mating faces, walls, ribs and reliefs; every component and every feature the route needs | supplier model, or a PROXY with sourced dimensions |
| DETAIL | fit, fastener, insert and retention features; chamfers, rounds, text and finish features | as bought: supplier model, or a PROXY whose `bounds` are sourced |

- Relief and clearance hold over the full joint `limits`, never only at the nominal pose.
- A part whose interface changes returns to PARTS form for the affected features.
- A purchased part that is only a guessed shape blocks every stage, LAYOUT included: source its model or bounds first.

## design-parts

Input: the approved LAYOUT version (assembly, interfaces, part records and models), the approved selection,
requirements and check plan.

1. Group PNs that share one kernel and interact; one `cad-part-designer` unit per group (cad-flow
   references/worker-brief.md):
   - `task`: the PARTS form of each PN, the kernel, and each PN's revision (unchanged, or the next draft `X<n>` for a
     part that changes after an approved version);
   - `subjects`: the PNs and their components;
   - `expected.outputs`: `hw/parts/<PN>/part.yaml`, `hw/parts/<PN>/model.py` for an AUTHORED or PROXY model, and each
     retained `hw/parts/<PN>/sources/<name>` the model reads;
   - `expected.checks`: `geometry-build`, `geometry-readback`, `geometry-export` and `geometry-identity`, plus
     `geometry-manufacture` for a MAKE or MADE_TO_DRAWING part (UNIT);
   - `boundary.locked`: interfaces, assembly, requirements, selection, check plan and every other PN.
2. One `cad-sourcing-researcher` unit for each purchased family, stock, supplier model or capability a route needs
   that no retained source covers (cad-select).
3. A revision change, an interface or assembly proposal, or a consequential UNKNOWN a delivery reports goes through
   cad-flow references/delegation.md "Integrate"; an orientation or support proposal becomes an ORIENTATION or
   SUPPORT decision (cad-flow references/gates.md "Decisions outside gates").

Output: one merged part record and model per PN; the assembly occurrences carry the revisions the briefs named.

## detail-parts

Input: the approved PARTS version, every decision approved since, the check plan and the requirements.

1. Lead first, in `hw/interfaces.yaml`: each critical interface's `fit` (kind, designation and `state` from a
   retained standard or supplier source), `variation`, `datums` and `acceptance`, KNOWN or typed UNKNOWN. Limits and
   stacks are cad-verify work (references/fits.md).
2. One detail unit per PN group as in `design-parts`, with the DETAIL form and these additions:
   - `task`: fits, tolerances, datums, fasteners, inserts, retention, support, setup, finish, inspection,
     manufacturing state and drawing intent; each orientation or support decision APPROVED since PARTS, with its DR;
   - `resources.skills` also lists cad-release (cad-flow references/agent-roles.md);
   - `expected.checks`: as `design-parts`, plus each check-plan check whose subject is one of the PNs.
3. Merge, then validate every part record → exit 0.

Output: detailed part records and models whose drawing intent states REQUIRED or NOT_REQUIRED for every PN.

## Design one part (cad-part-designer)

Read only the brief inputs, from their copies under `.cad-work/<unit>/input/`. Write each delivered file at
`.cad-work/<unit>/output/<its hw/ target path>`. Interfaces, assembly and other PNs are locked: a needed change is a
proposal under `output/proposals/`.
A lite part unit follows [Lite part](#lite-part).

1. Identity: `PN`, `rev` from the brief; `title`; `sourcing` from the approved candidate.
2. `model` (cad-model references/model-contract.md):
   - `AUTHORED`: `kernel`, `path: hw/parts/<PN>/model.py`, `frame: PART_LOCAL`;
   - `SUPPLIER`: the retained supplier file as `file {path, sha256}`, its `source` and `frame`;
   - `PROXY`: a bounded stand-in for a purchased part, with `kernel`, `path`, `frame`, the sourced `bounds` it
     encloses and the `reason`;
   - `NONE`: no geometry, with the `reason` (a consumable such as adhesive or grease).
3. `transparency` from the requirements; an optical need follows cad-select's optical rule.
4. `components[]`: every part with an AUTHORED or PROXY model lists each disconnected solid, one entry for a single
   solid: `id`, a `witness` (an interior point in the native frame) or a `locator` (a semantic feature),
   `material_profile`, `multiplicity` and `functions`.
5. `intent`, all from the approved candidate and the brief inputs; anything else is a typed UNKNOWN:
   - `functions`, `material_profile`, `service` (requirements, loads, environment, duty, life);
   - `critical_surfaces[]`: one per datum, bore, seat, journal, running, sliding, flank, cam, seal, fastener,
     locating or optical surface, with `feature`, `role`, `interfaces`, `requirements`, `criteria`, `finish`,
     `inspection` and `state`;
   - `route[]`: the candidate's operations in order, each with `id`, `kind`, `capability` (a profile id or a typed
     UNKNOWN), `provider`, `components`, `depends_on`, `targets`, and as the kind needs them `setup`, `orientation`
     (status `PROPOSED` until its decision is APPROVED), `supports`, `allowances`, `post_processing`, `inspection`;
     a `PURCHASED` operation carries `purchase`;
   - `selection`: the approved candidate id; `rationale.decision`: the APPROVED INTENT decision.
6. `manufacture` for a MAKE or MADE_TO_DRAWING part: `SAME`, or `DIFFERENT` with `state`, `reason`,
   `volume_tolerance` and `decision`, the APPROVED DR for that state difference. Without one, deliver the state as a
   proposal and stop.
7. `procurement` for BUY and MADE_TO_DRAWING: identity, specification, supplier, ratings, quantity and sources;
   availability, price and lead time stay UNKNOWN until sourced and dated.
8. `drawing`, always with `status` and `reason`: before DETAIL `UNKNOWN`; at DETAIL `REQUIRED` with `datums`, `views`
   and `characteristics` (each `id`, `feature`, `kind`, `criterion`, `datums`, `inspection`, `function`), or
   `NOT_REQUIRED` (cad-release references/drawings.md decides which).
9. Features: every interface member, critical surface, datum, component locator, route target and drawing
   characteristic names a semantic feature that the model's `features` witnesses under the same id.
10. Detail rules (DETAIL):
    - model a hole or shaft at the fit the interface states, plus a route allowance only from a capability profile
      calibration; without one the allowance is a typed UNKNOWN;
    - size clearance holes, counterbores, threads, thread preparation and insert bores from the retained standard or
      supplier datasheet, cited in the criterion `source`;
    - realize each operation's `retention` with a modelled feature (groove, snap, thread, seat);
    - in an additive operation's `orientation`, no feature lies below the face its `setup.datums` names first.
11. Write `model.py`, `build.py` and `check.py`, run them and deliver per cad-model; validate `part.yaml` against the
    `part` root → exit 0.

## Lite part

A lite part unit ([Lite layout](layout.md#lite-layout)) runs "Design one part" with the changes below; the part
records no material, route, sourcing, drawing or manufacturing state. The model program follows
[cad-model Lite model](../../cad-model/references/model-contract.md#lite-model).

- Step 1: `PN`, draft `rev` and `title` from the brief, plus `build_mode: LITE`; no `sourcing`.
- Step 2: `AUTHORED` only.
- Step 4: each component without `material_profile`.
- Step 5: `intent.functions` only, each function with the REQ ids it serves in `requirements`.
- Steps 3, 6, 7, 8 and 10 do not apply: omit `transparency`, `manufacture`, `procurement`, `drawing` and
  `mass_target`.
- Form: the LAYOUT, LITE build row of [Fidelity by stage](#fidelity-by-stage). Model each feature the brief's
  requirements name at its requested dimension, and witness it under its semantic id (step 9). A requested feature
  that is left out or replaced by an envelope is a MAJOR finding (cad-review rubric), never a delivered approximation.
