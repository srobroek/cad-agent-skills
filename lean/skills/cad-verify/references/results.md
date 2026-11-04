# Check results

Owner: cad-verify. How a producer fills one check result (root `check-result`; field list in
[cad-record contract index — Check result envelope](../../cad-record/references/contract-index.md#check-result-envelope-check-result)).
Paths, bindings and the hash graph are cad-record's; acceptance and publication are cad-flow's
([worker delivery](../../cad-flow/references/worker-delivery.md), [stages — run-checks](../../cad-flow/references/stages.md#run-checks)).

## Implementation kind

| Kind      | Who                                                                                                       | Contents                                                                                                                 |
| --------- | --------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| `COMMAND` | every family except `review.mechanical_dfm`; the lead's result of any family stopped at acceptance or of a lost or stalled unit ([families](families.md)) | `source_files` (the retained programs under `source/`), `commands` (argv, cwd, exit code, published `stdout` and `stderr`), pinned `packages`, `platform` |
| `REVIEW`  | the `review.mechanical_dfm` reviewer, always                                                              | `inspected`: every file read with its sha256 copied from the brief inventory; no commands, no artifacts                  |

1. A COMMAND producer writes its own program as a PEP 723 script, a unit at `.cad-work/<unit>/check.py` and the lead at
   `.cad-work/lead/checks.py`, and locks and runs it with the
   [cad-flow agent-roles uv commands](../../cad-flow/references/agent-roles.md#uv-commands). A program that reads record
   YAML parses it with the [pinned loader](../../cad-record/references/validation.md#yaml-loader). Program and lock stay at
   the unit root, never delivery outputs ([cad-flow agent-roles — Every role](../../cad-flow/references/agent-roles.md#every-role)
   step 6); at merge the lead copies them into the result bundle's `source/`
   ([cad-flow worker delivery — merge-deliveries](../../cad-flow/references/worker-delivery.md#merge-deliveries) step 3):
   `hw/checks/vN/<check>/<attempt>/source/`, or for `drawing.coverage` `hw/drawings/vN/<SUBJECT>/source/`
   ([cad-release drawings — drawing.coverage](../../cad-release/references/drawings.md#drawingcoverage)). The lead retains
   its own program and lock as `source/checks.py` and `source/checks.py.lock` of every result it produces. A command's
   `stdout` and `stderr` name published logs, never a scratch path: in a check bundle `stdout.txt` and `stderr.txt` beside the
   result's target (layout `check.stdout`, `check.stderr`), as `source_files` name `source/<file>`; any other result names
   only a log its own bundle publishes. The lead copies the delivered logs there at merge
   ([cad-flow worker delivery — merge-deliveries](../../cad-flow/references/worker-delivery.md#merge-deliveries) step 3).
2. A checker may start from the retained `source/check.py` and lock of the latest accepted result of the same family (one
   a stage verdict lists) for a re-attempt, a rework or a later stage, when its brief's input inventory lists both. The
   copy is its own program: it re-checks the copy against the current method and its brief, relocks and re-runs it, and
   the new result's `source_files` hash the program it ran.
3. The checker never runs or imports a producer's `build.py` or `check.py` as its method; producer exports are inputs only
   ([cad-record validation — Semantic check](../../cad-record/references/validation.md#semantic-check) item 18).
4. A program that did not finish leaves no PASS: its obligations are UNKNOWN `TIMEOUT` or `NOT_RUN`, and a result that was never written is
   `MISSING_RESULT` at acceptance.
5. A REVIEW result that lists commands, or a reviewer that wrote any file or ran anything other than read-only inspection commands (the lead
   compares its tree digest before dispatch and after return; per-host reading rule:
   [cad-flow agent roles — Roles](../../cad-flow/references/agent-roles.md#roles)), fails acceptance; the lead never edits the reviewer's
   bytes.

## Outcomes

One outcome per obligation in `scope.expected_obligations`, nothing more (cad-record semantic check 8).

| Outcome          | Needs                                                                                                                                            |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| `PASS`           | a measured `value` with units, the check-plan `criterion` unchanged, `method`, `evidence` files, `sampling`, covering the whole obligation scope |
| `FAIL`           | the same fields, with the value outside the criterion                                                                                            |
| `UNKNOWN`        | `reason` and the `unknown_code` the table below gives                                                                                            |
| `NOT_APPLICABLE` | `reason` and `reviewed_by` (role and unit); never for a missing method or input                                                                  |

A result never writes `WAIVED`: the producer records what it observed, and the stage verdict maps an approved waiver
([cad-flow stages — write-verdict](../../cad-flow/references/stages.md#write-verdict)).

| Situation                                                      | `unknown_code`               |
| -------------------------------------------------------------- | ---------------------------- |
| stopped by a FAIL or UNKNOWN obligation it depends on          | `NOT_RUN`                    |
| part of the scope measured, or `coverage_kind: PARTIAL`        | `INCOMPLETE`                 |
| an input, case, criterion or method changed since the evidence | `STALE`                      |
| a file, fact, model or source the method needs is absent       | `MISSING_INPUT`              |
| no method, tool, solver or capability exists for it            | `UNSUPPORTED_METHOD`         |
| the deadline ended the run                                     | `TIMEOUT`                    |
| a criterion, property or condition it needs is UNKNOWN         | `REQUIRED_UNKNOWN`           |
| only a physical test can establish it                          | `PHYSICAL_EVIDENCE_REQUIRED` |

Result: FAIL when any outcome is FAIL; else UNKNOWN when any is UNKNOWN or `complete` is false; else NOT_APPLICABLE when all are; else PASS.
`complete` is true only when every expected obligation has an outcome from a finished run.

## Coverage

- `FULL`: every item of a finite scope was evaluated: every pair, feature or contributor, and every discrete state a criterion is stated at
  (`ENDPOINTS_EVENTS`). Poses that sample continuous travel are never `FULL`.
- `SAMPLED`: poses that sample continuous travel, whatever the sampling kind (`GRID`, `CONSTRAINED_PATH`, `SAMPLED`, or the stepped
  approach and insertion of `motion.access` and `assembly.operations`); the declared sampling ran completely; a PASS holds for the sampled
  states only, and `scope.unsampled` lists every interval between samples. A criterion stated over continuous travel is UNKNOWN
  `INCOMPLETE` over those intervals unless a computed bound covers them.
- `PARTIAL`: any scope item missing; the result is never PASS.
- Excluding a pair, pose, component or feature from the scope is a `SCOPE_GAP`, never a smaller PASS.
- `FULL` on continuous travel violates
  [cad-record validation — Semantic check](../../cad-record/references/validation.md#semantic-check) item 8.

## Facets

Facets are independent: one never implies another, and an outcome claims only its own.

| Facet                          | Established by                                                                                                     | Never established by                                                                        |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------- |
| `CAD_VALID`                    | readback of the version geometry and a measurement against the criterion                                           | a render, preview or the producer's own log                                                 |
| `GEOMETRICALLY_MANUFACTURABLE` | geometry measured against a process criterion from a sourced capability profile                                    | a generic rule of thumb, machine resolution or a general tolerance label                    |
| `PROCESS_PLANNED`              | a complete recorded route: identified capability, approved orientation, setup, supports, allowances and inspection | a template capability or an envelope auto-fit                                               |
| `SIMULATION_SUPPORTED`         | a hand calculation or FEA case with sourced inputs, validity limits and, for FEA, convergence                      | density as strength, typical as allowable, static analysis for fatigue, creep, wear or life |
| `PHYSICALLY_VALIDATED`         | "Physical evidence" below                                                                                          | geometry, simulation, a supplier datasheet, a print that looks right, or a planned test     |

## Physical evidence

The `physical.test` method. A PASS needs, retained under `artifacts/` with role `SOURCE_EVIDENCE`:

1. an as-built specimen of this version: PN, part revision, material lot or profile, route and process state;
2. the protocol and setup of the check-plan entry, run on that specimen;
3. a calibrated instrument whose calibration source is unexpired at the test date;
4. the measured results with units, date, scope, conditions and the responsible person;
5. the criterion met over the tested configuration only.

Before RELEASE the obligation is future: PLANNED in the verdict, never an outcome. At RELEASE, no qualifying test is UNKNOWN
`PHYSICAL_EVIDENCE_REQUIRED`; only an approved `UNVALIDATED_TRIAL` waiver the service class permits
([check-plan — Evidence by service class](check-plan.md#evidence-by-service-class)) maps it to WAIVED in the verdict, and it is never PASS.
A test of a changed design no longer applies
([cad-flow change impact — Stale evidence](../../cad-flow/references/change-impact.md#stale-evidence)).

## Findings

- Every FAIL or UNKNOWN outcome has at least one finding with id `<family>:<anchor>:<locus>`
  ([cad-record record — IDs](../../cad-record/references/record.md#ids)); repeated symptoms with one evidenced cause are one finding.
- Fields: subject, severity ([cad-review rubric — Finding severity](../../cad-review/references/rubric.md#finding-severity)), outcome,
  measured value or the reason it is unknown, criterion and its source, method, uncertainty, consequence, remedy with alternatives, and
  `recheck` (families, earliest stage, scope).
- A finding never weakens, reinterprets or replaces the criterion; a better criterion is a change proposal.

## Freshness

A result is stale under [cad-flow change impact — Stale evidence](../../cad-flow/references/change-impact.md#stale-evidence) and reruns; it
is carried into a later verdict only under [Carry-forward](../../cad-flow/references/change-impact.md#carry-forward). A producer never
relabels, edits or re-dates a published result.
