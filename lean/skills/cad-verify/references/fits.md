# Fits and stacks

Owner: cad-verify. Methods for `fit.limits` and `fit.stack`. Interface fit and variation fields are the
[cad-record interfaces prose root](../../cad-record/references/prose-contract.md#interfaces); STACK cases are the
[cad-record check-plan prose root](../../cad-record/references/prose-contract.md#check-plan).

## fit.limits

Per interface or feature that states a fit or tolerance class:

1. Take the limits from the standard of the product `regime` (ISO: ISO 286-1 and ISO 286-2; ASME: ASME B4.2) as a retained, cited table
   source. A size, grade or deviation outside the retained table is UNKNOWN `UNSUPPORTED_METHOD`, never extrapolated or recalled.
2. Compute hole and shaft limits in the interface's state (AS_MANUFACTURED, AFTER_FINISH or ASSEMBLED), add every stated coating, thermal
   and moisture effect with its source, and derive minimum and maximum clearance.
3. Classify: CLEARANCE when minimum clearance ≥ 0, INTERFERENCE when maximum clearance ≤ 0, otherwise TRANSITION. A class different from the
   interface's intent is FAIL.
4. Compare each functional criterion (running clearance, retention interference) at both limits.
5. Measure the model's semantic feature size and state whether the model is at nominal, at a limit or at the manufacturing-state size; a
   mismatch with the declared nominal is FAIL.
6. A fit code is no capability evidence: the operation forming the feature needs that tolerance inside its `capability.route` profile; that
   obligation is separate.

## fit.stack

Per STACK case:

1. A one-dimensional signed chain along one stated direction: each contributor's nominal, tolerance interval `[lower, upper]`, state and
   source; every stated coating, adhesive, thermal, moisture, play, wear and deformation effect is its own contributor with a source.
2. Worst case (default): a `+1` contributor adds `[nominal + lower, nominal + upper]`, a `−1` contributor adds
   `[−(nominal + upper), −(nominal + lower)]`; the result interval is the sum. Evaluate the criterion at both ends.
3. A nominal result that passes while an end fails is FAIL: nominal +0.20 mm with a minimum of −0.10 mm misses a clearance criterion of ≥ 0.
4. `JUSTIFIED_RSS` only with the case's `rss` block and its DR: stated distribution, independence or correlation, centering, sigma,
   confidence and process capability. Publish the worst-case and RSS intervals separately; the calculation never establishes independence.
5. A contributor without a tolerance or source is UNKNOWN `REQUIRED_UNKNOWN`.
6. DATA artifact: every contributor, sign, interval, effect and the result interval.
