# Process checks

Owner: cad-verify. Methods for `capability.route`, `optical.route`, `purchase.identity` and the 13 `process.*` families. Material and route
choice is [cad-select's](../../cad-select/SKILL.md); route fields are the [cad-record part root](../../cad-record/schemas/part.schema.json)
(`intent.route[]`, operation `capability`, `setup`, `orientation`, `supports`, `allowances`, `inspection`, `purchase`).

## Route kind to family

| Route kind                                                                 | Family                                      |
| -------------------------------------------------------------------------- | ------------------------------------------- |
| `FDM`                                                                      | `process.fdm`                               |
| `RESIN`                                                                    | `process.resin`                             |
| `CNC`                                                                      | `process.cnc`                               |
| `LASER`                                                                    | `process.laser`                             |
| `WATERJET`                                                                 | `process.waterjet`                          |
| `PLASMA`                                                                   | `process.plasma`                            |
| `SHEET_FORMING`                                                            | `process.sheet_forming`                     |
| `JOINING`                                                                  | `process.joining`                           |
| `CASTING`                                                                  | `process.casting`                           |
| `MOULDING`                                                                 | `process.moulding`                          |
| `EXTRUSION`                                                                | `process.extrusion`                         |
| `DRILLING`, `REAMING`, `TAPPING`, `FINISHING`, `COATING`, `HEAT_TREATMENT` | `process.secondary`                         |
| `OTHER`                                                                    | `process.other`                             |
| `INSPECTION`                                                               | no process family; `capability.route` only  |
| `PURCHASED`                                                                | no process family; `purchase.identity` only |

A part with several operations gets one obligation per operation and feature for each family; a purchased part or a CNC-only part gets no
FDM obligation.

## Rules for every process family

1. Criteria come from the check-plan criterion and the operation's sourced capability profile: its calibrations, material approvals,
   envelope and demonstrated limits. No universal wall, overhang, bridge, hole, kerf, radius, draft or tolerance limit exists; a criterion
   the profile does not give is UNKNOWN `REQUIRED_UNKNOWN`.
2. A template capability, a machine resolution or a general tolerance label is no criterion.
3. Measure the version's manufacturing-state geometry in the declared orientation, setup and support plan; a plan item the criterion depends
   on that is missing makes that obligation UNKNOWN `REQUIRED_UNKNOWN`.
4. Process obligations establish GEOMETRICALLY_MANUFACTURABLE; `capability.route` establishes PROCESS_PLANNED; coupons, print trials and fit
   trials are `physical.test` obligations, never process outcomes.
5. Each violated item is a finding whose locus is the item code in brackets below; the obligation is FAIL.
6. Access proven in CAD is not a toolpath, slicer or CAM result; say so in `limitations`.

## capability.route

Per made operation (every route kind except PURCHASED):

1. `capability` names a real profile, not a template, whose provider, machine or operator is identified.
2. The profile approves the operation's stock or material state, and the component fits its envelope.
3. Each operation criterion is within the profile's demonstrated capability for that feature and state.
4. Calibrations the criterion relies on are unexpired at the verdict time.
5. The route is ordered and complete: `depends_on`, input and output states, and the plan items the kind needs (orientation for FDM and
   RESIN; setup, datums and workholding for CNC; stock identity and thickness for cutting and forming; allowances where finishing follows;
   inspection for every critical surface).

An UNKNOWN capability, provider or calibration is UNKNOWN `REQUIRED_UNKNOWN`; a criterion outside the demonstrated capability is FAIL.

## optical.route

Per component with `transparency` TRANSPARENT or TRANSLUCENT, or a requirement for light transmission. The selection rule is cad-select's
(SKILL.md MUST on transparency); this check verifies the record against it.

1. FAIL [`fdm_optical`] when any operation forming the optical region is FDM, whatever the filament's name or colour.
2. FAIL [`resin_optical`] when any resin print forms the optical region, whatever the resin's name, clarity or qualification.
3. PASS needs a purchased transparent part or identified transparent stock: traceable grade, thickness, supplier and sourced optical
   criterion, on a route whose capability approves that stock.
4. Actual transmission of the made part is a `physical.test` obligation; until it exists the optical requirement's PHYSICALLY_VALIDATED
   facet is PLANNED or UNKNOWN.

## Purchased

`purchase.identity`, per BUY part and PURCHASED operation:

1. Identity, manufacturer or supplier, catalogue number, standard, dimensions, material, rating, mount, retention and availability are KNOWN
   with a retained source, or typed UNKNOWN.
2. The interface the item meets matches the sourced dimensions and ratings; a bounded proxy model is labelled as such.
3. No made-part process check applies; an as-bought fidelity the check plan needs from a placeholder model is UNKNOWN `MISSING_INPUT`.

## process.fdm

Orientation and export pose approved [`orientation_unapproved`, `pose_unmeasured`]; first layer, bed contact and stability [`first_layer`,
`stability`]; envelope [`envelope`]; structural walls distinguished from tips, chamfers and decorative features [`wall`]; small features and
hole axes [`small_feature`, `hole_fit`]; bridges, overhangs, islands and layer direction against load [`overhang`, `layer_direction`];
support start, contact and removal access [`support_unplanned`]; every bore, seat, journal, gear, cam, press fit, insert and boss;
calibrated shrinkage, warp, hole compensation, elephant foot, seams and stair-stepping [`warp`, `elephant_foot`].

- A reachable support on a non-critical surface passes. A support contacting a critical surface is FAIL [`support_on_critical`] unless the
  route finishes that surface with an allowance, access and inspection; a soluble support alone does not make it pass.
- A missing or wrong support plan is UNKNOWN or FAIL, never "no supports".

## process.resin

The FDM orientation and support items, plus drainage and venting, trapped resin [`trapped_volume`], cleaning, cure and post-cure
[`post_cure_unplanned`], distortion, stability, ageing and brittleness [`ageing`].

## process.cnc

Stock identity, form, grade, dimensions and machinability [`stock_unknown`, `stock_material`]; axes, setups, datums, fixtures, workholding
and clamp distortion [`axes_unknown`, `workholding_unknown`, `datums_unknown`, `clamp_on_critical`]; cutter and holder diameter, reach and
collision, internal radii, deep pockets, undercuts and unsupported features [`tools_unknown`, `reach`, `internal_radius`, `undercut`]; drill
and tap access, size, engagement, blind depth and breakthrough [`hole_access`, `hole_depth`, `breakthrough`]; thin walls, burrs, finishing
allowance and inspection access [`finishing_allowance`, `inspection_access`]; demonstrated tolerance and finish [`tolerance`, `finish`,
`precision_uncalibrated`].

- A holder collision or a cutter larger than an internal radius is FAIL [`reach`, `internal_radius`].
- A missing clamp or setup plan is UNKNOWN [`workholding_unknown`].

## process.laser

Operation mode cut, score or engrave kept distinct (an engrave is never a precision pocket or bore); exact stock, coating and thickness
against the machine, power, gas, extraction and approval [`material_unidentified`, `thickness_unknown`, `extraction`]; kerf and
compensation, taper, minimum hole and web, edge quality, heat-affected zone, burning, distortion and flatness [`kerf_uncalibrated`,
`min_hole`, `min_web`]; stock thickness variation, nesting, grain, loose cut-outs and micro-tabs; tab and slot fit with thickness, kerf and
coating ranges [`tab_slot_fit`]; secondary operations and assembled rigidity.

- PVC and other chlorine-releasing stock is FAIL with CRITICAL severity [`material_unsafe`]; an unidentified plastic or coating, or absent
  extraction or machine approval, is UNKNOWN with CRITICAL severity.
- Tab and slot fit is evaluated at the worst case of stock thickness, kerf and coating ranges, not the nominal: a nominal +0.10 mm clearance
  that reaches −0.15 mm at the worst case is FAIL; a passing interval still needs a fit coupon (`physical.test`).

## process.waterjet

Its own profile and criteria, never inherited from laser: kerf, taper [`taper_uncalibrated`], minimum features, edge finish
[`edge_finish_uncalibrated`, `edge_finish`], abrasive and stock effects, secondary finishing.

## process.plasma

Its own profile and criteria, never inherited from laser: kerf, taper, minimum features, heat-affected zone and distortion, cut-datum
flatness, dross and secondary finishing.

## process.sheet_forming

Thickness, bend radius, bend allowance and springback calibrated for the stock [`bend_uncalibrated`, `bend_allowance_uncalibrated`,
`springback_uncalibrated`, `bend_radius`]; reliefs; hole-to-bend distance [`hole_to_bend`]; tooling access and bend sequence
[`sequence_unplanned`]; grain direction; coating damage; folded tolerances and load paths.

## process.joining

Joint method and filler or fastener identity; access for the tool, torch or press; fit-up and gaps; distortion and heat input; sequence;
strength evidence for the joint as `load.*` or `physical.test`, never from the parent material.

## process.casting

Draft and pull direction [`pull_unknown`, `draft`]; wall transitions; shrinkage [`shrinkage_unknown`]; parting line; undercuts [`undercut`];
tooling and its cost against quantity [`tooling_unplanned`]; defect evidence [`defect_evidence`]; machining allowance and secondary
machining of critical features [`machining_allowance`, `secondary_missing`].

## process.moulding

The casting items for the moulding process and material, plus gate, weld-line and sink locations from the tooling source.

## process.extrusion

The profile is prismatic along the extrusion axis [`not_prismatic`]; profile tolerance from the supplier source; cut-to-length and every
secondary operation present [`secondary_missing`].

## process.secondary

Per DRILLING, REAMING, TAPPING, FINISHING, COATING or HEAT_TREATMENT operation: datum transfer from the input state; targets, access and
reach; tool size and depth for drilling; thread engagement for tapping; allowance and access for reaming and finishing; coating thickness
and its compensation on fits; heat-treatment distortion and allowance; inspection of the result. A missing ream allowance or access, or an
uncompensated coating on a fit, is FAIL.

## process.other

Needs an explicit `description` and a check-plan criterion list for that process; without them every obligation is UNKNOWN
`UNSUPPORTED_METHOD`. Never substitute another process's criteria.
