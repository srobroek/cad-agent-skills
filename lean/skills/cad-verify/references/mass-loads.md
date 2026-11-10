# Mass and loads

Owner: cad-verify. Methods for `mass.properties` and `load.hand`. MASS and LOAD cases are the
[cad-record check-plan prose root](../../cad-record/references/prose-contract.md#check-plan); material facts and bases are the
[cad-record material-profile root](../../cad-record/schemas/material-profile.schema.json), selected under
[cad-select materials](../../cad-select/references/materials.md).

## mass.properties

Per MASS case:

1. Per made component: volume of the version readback in the state the case names × the case density, converted explicitly
   (`mm3 × g_cm3 / 1000 = g`). A printed part's density is that of its printed state; a bulk density for a printed part is an ESTIMATE with
   its bound.
2. Per purchased item: a measured or sourced mass; a proxy mass is labelled as one.
3. Multiply by multiplicity and place each centre of mass by its occurrence transform in the assembly frame.
4. A missing mass or density is UNKNOWN, never zero; the aggregate is then UNKNOWN `REQUIRED_UNKNOWN`.
5. Propagate every ESTIMATE and ASSUMED bound into a mass interval and a centre-of-mass region.
6. Stability criteria (tipping, support polygon) use the worst credible mass distribution and the worst pose of the case's MOTION evidence;
   friction or contact assumptions the criterion needs are sourced or UNKNOWN.
7. Outputs: the case's `MASS`, `CENTRE_OF_MASS` and `INERTIA`, each with units and its interval.

## load.hand

Per LOAD case with `method: HAND`:

1. Free-body diagram of the case: every load (kind, magnitude, direction, application), boundary condition and reaction; force and moment
   balance checked numerically and recorded.
2. Each formula cites its source (document, edition, equation) and the validity limits the case states; the model stays inside them or the
   outcome is UNKNOWN `UNSUPPORTED_METHOD`.
3. Units are converted explicitly; mixed units without a conversion are a FAIL of the method.
4. Material values come from the case's profile at the `material.basis_required` this step sets:
   - `ALLOWABLE`, with authority, conditions and derivation, for a strength, fracture, buckling, fatigue or creep criterion and for every
     criterion of a safety requirement;
   - `SUPPLIER_TYPICAL` or `MEASURED` only for a static stiffness or deflection criterion of a non-safety requirement, on the component's
     grade, form, orientation and process state, with the source's stated uncertainty carried into the outcome: PASS only when the whole
     range meets the criterion, and a value without a stated uncertainty is UNKNOWN `REQUIRED_UNKNOWN`;
   - never a density or a value of another grade, form, orientation or process state: such an outcome is UNKNOWN `REQUIRED_UNKNOWN`.
5. Evaluate every listed failure mode. A static method covers static modes only: CYCLIC, SUSTAINED, IMPACT and VIBRATION loads need a
   qualified method and data for fatigue, creep, impact or resonance, else UNKNOWN `UNSUPPORTED_METHOD`.
6. A demand above capacity is FAIL with the margin: a 0.1883 N·m holding torque against a 0.24586 N·m demand fails.
7. The facet is SIMULATION_SUPPORTED; a hand result is never physical evidence.
