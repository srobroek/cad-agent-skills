# Rubric

Owner: cad-review. The one grading rubric for every worker delivery and every review: finding severity and the verdict `PASS|MINOR|MAJOR`.
The reply format is [cad-flow agent roles — Reply](../../cad-flow/references/agent-roles.md#reply); the finding fields are
[cad-verify results — Findings](../../cad-verify/references/results.md#findings); what a verdict does to a stage is
[cad-flow stages — write-verdict](../../cad-flow/references/stages.md#write-verdict) and
[cad-flow change impact — Findings](../../cad-flow/references/change-impact.md#findings).

## Finding severity

| Severity   | When                                                                                                                                                                                                                                                                                            |
| ---------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `CRITICAL` | a safety or regulatory requirement fails or is unestablished; a proven unsafe failure; unsafe or unidentified stock, coating or process conditions (chlorine-releasing laser stock, absent extraction)                                                                                          |
| `MAJOR`    | a due requirement or obligation is FAIL, UNKNOWN or not run; a result is missing, schema-invalid, stale, out of scope or bound to the wrong input; a collision within required travel; an unsafe load path or retention; a feature the route cannot make; a boundary violation; unusable output |
| `MINOR`    | a bounded, non-safety discrepancy against a non-mandatory criterion, or an advisory, while every due acceptance criterion is met                                                                                                                                                                |
| `INFO`     | an observation with no criterion at stake                                                                                                                                                                                                                                                       |

A mandatory UNKNOWN is never MINOR. A planned fix, an old finding or a waiver request lowers no severity.

## Verdict

| Verdict | When                                                                                                                                                                                                                                                                                                                    |
| ------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `MAJOR` | any CRITICAL or MAJOR finding; any due, non-waived FAIL, UNKNOWN or not-run obligation in the graded scope; a missing or invalid result; a write outside the brief's `boundary.writable`, a reviewer command beyond read-only inspection, or an edited installed file; an unsafe or unqualified claim; output the brief cannot use |
| `MINOR` | every due acceptance criterion met, no CRITICAL or MAJOR finding, and at least one MINOR finding                                                                                                                                                                                                                        |
| `PASS`  | the assignment complete; every due criterion evidenced; coverage complete ([cad-verify results — Coverage](../../cad-verify/references/results.md#coverage): FULL, or SAMPLED with every unsampled interval listed and a computed bound wherever a continuous-travel criterion spans one); no unresolved finding within the approved readiness scope |

A PASS states every future obligation as PLANNED and every permitted waiver as WAIVED; neither is evidence, and neither is physical
validation.

## Per role

| Role                                        | The verdict grades                                                                                                                                                                                     |
| ------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `cad-concept-designer`, `cad-part-designer` | the delivery against its brief; only PASS is ready                                                                                                                                                     |
| `cad-sourcing-researcher`                   | the research scope of the brief; a PASS finds a conditional option, never a final feasibility or selection                                                                                             |
| `cad-checker`                               | the checked subject by its results: PASS when every result is PASS or reviewed NOT_APPLICABLE and no finding is above INFO; MINOR when only MINOR findings remain; MAJOR on any FAIL or UNKNOWN result |
| `cad-mech-dfm-reviewer`                     | the reviewed version, by [cad-review review — review.mechanical_dfm](review.md#reviewmechanical_dfm)                                                                                                   |

## Rules

MUST Grade by the worst finding and the worst due outcome in the graded scope. MUST Grade against the criterion as approved; a criterion you
think is wrong is a finding, never a reason to pass. NOT Grade PASS from missing, unread, UNKNOWN, PLANNED or WAIVED evidence, or from a
smoke, nominal or UNIT run. NOT Restate or change the verdict in prose: the first line of the reply carries it.
