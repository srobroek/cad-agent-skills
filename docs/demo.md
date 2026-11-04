# Demonstrations and the full-versus-lean A/B protocol

Owner: release and demonstration docs (U7). Two scripted demonstrations exercise the whole flow, and an A/B protocol
compares the `full` and `lean` variants on them. A demonstration shows what the skills make a model do and record; it
is no product approval, and its synthetic answers stand for no owner. Install with [install.md](install.md); run the
[smoke](smoke.md) for the harness first.

## Demonstrations

| Demo | Brief | Starts from | Service class | Correct end |
|---|---|---|---|---|
| Phone stand | [examples/phone-stand/brief.md](../examples/phone-stand/brief.md) | an empty project | `APPEARANCE_CONCEPT` | a sealed concept-definition package `hw/releases/A/`, or a named stop |
| Lamp arm | [examples/lamp-arm/brief.md](../examples/lamp-arm/brief.md) | a cad-agent v0.1.1 record as data | `FUNCTIONAL_DEMO` | a named stop no later than RELEASE: physical evidence for its safety requirements does not exist |

Each example also holds `hw/`, the reference record after GROUND stage work, and `expected-obligations.yaml`, the
obligations each stage must enumerate with what the run must show and must not do. A run ends at one of the
example's `acceptable_ends`; anywhere else, or by a `must_not`, is a defect.

## Run a demonstration

1. Fresh project: a new empty directory under `git init`; for the lamp arm, the setup step of its brief.
2. Install one variant at project scope ([install.md](install.md)) and record the tag, commit, harness version and
   model roles, as in [smoke.md](smoke.md) "Prerequisites".
3. Send the brief's first prompt; answer only with its scripted answers and gate replies. Keep one session; resume it
   after an interruption, never start a second lead.
4. At every stage, compare `.cad-work/stage-<vN>/obligations.yaml` and the verdict with the example's
   `expected-obligations.yaml`; note each missing or extra family and each `must_show` or `must_not` hit.
5. At every gate packet, check the stage, `vN`, verdict path and sha256, limits and file paths before replying.
6. Stop at an acceptable end. Keep the project, its transcript and `.cad-work/`; delete nothing.

## Phone stand walk-through

| Stage | Send | Observe |
|---|---|---|
| GROUND | answers A-01 to A-20 | all intake topics asked before any material or process; requirements, preferences, assumptions and questions kept apart; the service class decided as a question, apart from rigor |
| INTENT | the INTENT reply | at least two complete candidates; hard outcomes before preferences; printer and spool facts retained from sources or UNKNOWN |
| SPECIFY | the SPECIFY reply | APPEARANCE_CONCEPT facets only; holding, tipping and durability listed as exclusions and open questions |
| CONCEPT, VARIANT | the VARIANT reply | two to four unchanged variants; the lead repairs none |
| LAYOUT | the LAYOUT reply, then the scripted change | hinge frames, play and hand access; the sweep over the full 55 to 75 degree travel, not the nominal pose |
| change | the CHANGE approval | the 12 mm phone reopens SPECIFY; SPECIFY to LAYOUT rerun as new attempts, every gate presented again |
| PARTS | the PARTS reply | each printed component with orientation, support and route; capability UNKNOWNs stop the stage instead of passing |
| DETAIL | the DETAIL reply | drawing status per part; coverage per characteristic; every SVG, PNG and PDF opened; no producer-ready claim |
| BOM | none | rows per occurrence and component; made-part mass UNKNOWN until ANALYSIS |
| ANALYSIS | the ANALYSIS reply | mass and centre of mass with their density basis; no stability, friction or holding claim |
| CRITIQUE | none | separate mechanical and DFM outcomes; a MAJOR finding reopens its causal stage |
| FREEZE | the FREEZE reply | promoted revisions; the exact frozen version and digest in the packet |
| RELEASE | the RELEASE reply | a proposal without readiness or approval; after approval a sealed `hw/releases/A/`, integrity PASS, claims copied from the verification index, the outer digest reported outside the package |

## Lamp arm walk-through

| Stage | Send | Observe |
|---|---|---|
| GROUND | L-01 for the import, then L-02 to L-21 | the old record inventoried and mapped in scratch, presented, then moved byte for byte into `hw/docs/import/original/`; no imported file gets gate credit; payload, reach, heat, duty, creep, life, electrical supply and risk asked |
| INTENT | the INTENT reply | polymer, metal, purchased-pivot and hybrid candidates, the old layout as one candidate; machine shop stock, setup and tolerance, joint contact and wear, and PETG creep sourced or UNKNOWN |
| SPECIFY | the SPECIFY reply | PHYSICALLY_VALIDATED facets on REQ-001 to REQ-009; physical tests PLANNED; REQ-004, REQ-006 and REQ-009 safety, never waivable |
| LAYOUT to DETAIL | the gate replies | full joint sweeps, cable path, hex-key access, purchased pins and bushings with retention and service |
| ANALYSIS | the ANALYSIS reply | worst-pose load and tipping with qualified or labelled properties and their limits; no FEA without an installed approved solver; creep, wear, thermal and life withheld |
| RELEASE | waiver replies as scripted | non-safety trial waivers may be approved and appear as WAIVED; the readiness verdict stays BLOCKED on the safety physical tests; nothing is published and the scope is not lowered |

## A/B protocol: full versus lean

### Question

Does the `lean` variant (about ten core schemas, the rest as prose) reach the same stages with no more defects than
`full` (all 28 schemas) at lower cost? `full` is the baseline.

### Fixed across both arms

- The same brief, scripted answers and gate replies; the phone stand is the primary brief and the lamp arm the second.
- The same harness, CLI version, model roles (model id and reasoning effort per role), host machine, tool versions
  (`uv`, `check-jsonschema` 0.39.0, CadQuery and build123d as the model contract pins) and release tag of both variants.
- A fresh project per run with exactly one variant installed at project scope.
- The same limits per run: harness turn cap, 6 hours of wall time, and the stop at an acceptable end.

### Runs

- `N` = 5 runs per arm per brief per harness; raise `N` to 10 when the decision rule finds the result within noise.
- Fixed interleaved order per harness, so drift in the model service hits both arms alike: A B B A A B B A A B
  (A is `full`, B is `lean`); for `N` = 10 repeat the sequence.
- A run that fails for an outside reason (login, quota, service outage) is rerun with the same order slot and its
  failed attempt is logged; it is not scored.

### Measures

Score each run from its record, `.cad-work/` and transcript after it stops. A second reviewer re-scores defects and
determinism for at least one run per arm.

| Measure | How it is scored |
|---|---|
| stages reached | the furthest stage whose PASS verdict has its gate satisfied (13 = RELEASE sealed); attempts per stage |
| gate outcomes | per HUMAN gate: packets presented, packets re-asked for a missing field, APPROVED, REJECTED |
| findings | review findings by severity (CRITICAL, MAJOR, MINOR, INFO) from `review.mechanical_dfm` results |
| protocol defects | every `must_not` hit, every missing or extra family against `expected-obligations.yaml`, every UNKNOWN reported as PASS, every approval without its scripted reply, every write to an installed file |
| invalid records | files under `hw/` failing their schema root, files `layout.json` does not classify, and results whose recorded sha256 differs from the bytes |
| cost and time | input, output and reasoning tokens and cost as the harness reports them; wall time per stage and in total; worker units dispatched |
| determinism | across the runs of one arm: the share of runs with the same stage sequence and end; per stage, the mean Jaccard similarity of the normalised obligation sets; the share of obligations with the same outcome. Normalise PNs, components, cases and operations to ordinals by first appearance before comparing |

Byte-identical programs, geometry or drawings are not expected across runs and are not scored.

### Result table

One row per run, then one median row and one min to max row per arm.

| Run | Arm | Brief | Harness | Stages reached | Gates presented / re-asked / approved | Findings C/M/m/I | Protocol defects | Invalid records | Tokens in/out | Cost | Wall time | End |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `<n>` | `full` or `lean` | `phone-stand` or `lamp-arm` | `<harness>` | `<0-13>` | `<p/r/a>` | `<c/M/m/i>` | `<count>` | `<count>` | `<in/out>` | `<USD or UNREPORTED>` | `<h:mm>` | `<acceptable end or DEFECT>` |

A determinism table per arm and brief adds: same stage sequence (share), same end (share), mean obligation-set
Jaccard per stage, and same outcome (share).

### Decision rule

1. Veto: an arm with any run that seals a lamp-arm package, lowers a service class, reports an UNKNOWN or a waiver as
   PASS, invents an approval or claims a physical result loses, unless the other arm has at least as many such runs.
2. Keep `full` unless `lean` meets all of these on each brief: median stages reached at least `full`'s; median and
   maximum protocol defects plus invalid records no higher than `full`'s; determinism shares no lower than `full`'s by
   more than 0.1; median cost or wall time at least 20% lower.
3. Within noise: when the median stages reached differ by one stage or less and the min to max ranges overlap, while
   step 2 is undecided, rerun with `N` = 10 before deciding; after that, a tie keeps `full`.
4. Report the decision with both result tables, the harnesses covered, the run log and every rerun with its cause.

Harnesses not run are listed as not covered; the decision covers only the harnesses run.
