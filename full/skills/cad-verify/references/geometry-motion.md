# Geometry and motion checks

Owner: cad-verify. Methods for `geometry.interference`, `geometry.clearance`, `motion.sweep`, `motion.play`,
`motion.access` and `assembly.operations`. Inputs are the version's readback geometry and transforms (cad-model
references/exports.md), the assembly tree, joints and operations (cad-record `assembly` root), interfaces and the
MOTION cases of the check plan.

## Pair measurement

Used by every heading below; one OCCT path for CadQuery 2.8.0 (cadquery-ocp 7.9.3.1.1) and build123d 0.13.0
(cadquery-ocp-novtk 8.0.1.1.0); pass `obj.wrapped`.

1. Enumerate every occurrence pair in sorted order: repeated placements of one PN, fixed-joint and joint-connected
   pairs included. An AABB broad phase may skip only a pair whose boxes are farther apart than the largest
   clearance criterion that applies to it.
2. Minimum boundary distance with `BRepExtrema_DistShapeShape`; `IsDone()` false is a FAIL of the measurement,
   not a pass. A compound operand needs an explicit containment test per solid (`BRepClass3d_SolidClassifier`).
3. At a distance at or below 1e-6 mm, a non-destructive `BRepAlgoAPI_Common` with fuzzy value 1e-7 mm and an
   adaptive `BRepGProp.VolumeProperties_s`; shared volume above 1e-3 mm³ is interference.
4. A 0 mm reading is re-evaluated against the pair's bounds; an unresolved reading is FAIL.
5. Record per pair: occurrences, distance, shared volume, witness points, the pose and the excuse if any, in a DATA
   artifact; `pairs` counts and the tolerances above go in the outcome.

```python
from OCP.BRepExtrema import BRepExtrema_DistShapeShape  # cadquery-ocp 7.9.3.1.1 / 8.0.1.1.0
ext = BRepExtrema_DistShapeShape()
ext.LoadS1(a); ext.LoadS2(b); ext.Perform()  # a, b: TopoDS_Shape at their placements
distance_mm = ext.Value() if ext.IsDone() else None
```

```python
from OCP.BRepAlgoAPI import BRepAlgoAPI_Common  # same pins
common = BRepAlgoAPI_Common()
common.SetArguments(args); common.SetTools(tools)  # TopTools_ListOfShape of a and of b
common.SetNonDestructive(True); common.SetFuzzyValue(1e-7); common.Build()
```

## geometry.interference

1. `@assembly`: every pair at the nominal pose; PASS when no pair interferes outside a declared intended fit.
2. An interference fit, press fit or heat-set insert excuses overlap only for its declared occurrence pair and
   feature, stated by its interface; the same pair overlapping elsewhere is FAIL.
3. `@IF-###` for an intended interference: overlap lies only in the declared feature region; its magnitude is
   `fit.limits`' obligation.
4. No numerical tolerance beyond the measurement values above suppresses a hit.

## geometry.clearance

1. `@assembly`: minimum distance of every pair at the nominal pose against each clearance criterion an interface or
   requirement states; a pair with no stated criterion is reported, not graded.
2. `@IF-###[#<feature>]`: distance between the interface's semantic features (cad-model witnesses), never between
   face indices.
3. A nominal result says in `limitations` that tolerance extremes are `fit.stack` and play extremes are `motion.play`
   evidence.

## motion.sweep

Per MOTION case:

1. Poses from the case: every independent joint at its steps, endpoints and the events of the profile (limits, dwell,
   dead centres, extrema) always included. Dependent joints are derived from their coupling at every pose, never
   sampled on their own. The check enforces joint limits itself.
2. Sampling kind:
   - `GRID`: the Cartesian product of every independent joint's steps; one joint at a time with the others
     nominal is not a grid;
   - `CONSTRAINED_PATH`: the justified path the mechanism permits;
   - `ENDPOINTS_EVENTS`: only for a criterion stated at those states;
   - `SAMPLED`: any other declared set, with its justification.
3. At every pose apply "Pair measurement" to the case's pairs (`ALL` unless the case names them) in the tool and
   service state the case states. Dropping a pair that collides is a `SCOPE_GAP`.
4. Outcome value: the minimum clearance or the maximum shared volume over all poses, with its pose and pair; the
   DATA artifact holds every pose × pair.
5. Coverage `SAMPLED` lists the intervals between poses in `scope.unsampled`. A continuous-travel criterion over those
   intervals is UNKNOWN `INCOMPLETE` unless a computed bound covers each interval; a rate estimated from the samples
   is no bound.
6. A smoke run, a nominal pose or a render is never this evidence: a design that passed nominal checks has collided
   at 11 of 19 full-travel poses.

## motion.play

Per MOTION case with `play_extremes: true`, and per interface stating play or backlash:

1. Play is not part of a coupling law; take each stated play value and direction from its interface or joint.
2. At every pose of the case, displace each member with play to both extremes along each stated direction, alone
   and in the combination the case names, and apply "Pair measurement".
3. Play that matters to the criterion but is not stated is UNKNOWN `REQUIRED_UNKNOWN`, never zero.

## motion.access

Per assembly operation or service task with tool or hand access:

1. The tool or hand envelope comes from a sourced identity with dimensions; a tool named without dimensions is UNKNOWN
   `MISSING_INPUT`.
2. Build the assembly state before the operation: its predecessors installed, later occurrences absent; for a service
   task, the assembled state.
3. Move the envelope along the stated approach direction to engagement, sampled at a stated step, and apply "Pair
   measurement" against every occurrence present. Contact before engagement is FAIL.
4. Access checked only in the final assembled pose does not cover an earlier operation.

## assembly.operations

Per assembly operation:

1. Occurrences exist; `predecessors` form one acyclic order; `into`, `direction`, engagement, torque and retention
   are KNOWN with sources or typed UNKNOWN.
2. Insertion: sweep the inserted occurrence along `direction` from clear of the assembly to its placement against the
   occurrences installed before it; contact outside the declared fit region is FAIL.
3. A fastener or part whose access an earlier operation closes is FAIL (closed before install).
4. Every non-fixed occurrence has a retention; every service task has a removal path checked as in steps 2–3.
