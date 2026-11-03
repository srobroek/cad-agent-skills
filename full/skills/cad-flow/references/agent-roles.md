# Agent roles

Owner: cad-flow. The one contract for the five worker roles: duties, inputs, write boundary, outputs and reply.
Each native role file links here and repeats no rule. A worker reads this file, its brief and the skills its
brief lists; never cad-flow SKILL.md or another lead reference. Grading uses the rubric owned by cad-review:
`../../cad-review/references/rubric.md` (`skill://cad-review/references/rubric.md`).

## Every role

1. Read the brief at the absolute path in your task. Every resource is the literal absolute path or
   `skill://` URI in `resources`; never search the product directory or guess an install path.
2. LOAD the SKILL.md of each `resources.skills` entry the role table names; from cad-review read only
   `references/rubric.md` unless you are the reviewer.
3. Read the `input_inventory` files from their copies at `.cad-work/<unit>/input/<path>`, and the
   `boundary.read_only` paths; nothing else. A needed file outside them → stop.
4. Writers: take the tree digest (cad-record references/record.md "Inventory digest") before your first write
   as `boundary.before_sha256`, and again after your last write as `boundary.after_sha256`.
5. Do the role duty below; write only the `boundary.writable` paths of the brief.
6. Writers build and check with their own programs `.cad-work/<unit>/build.py` and `check.py`, using the run
   commands cad-model and cad-verify name. Each check in `expected.checks` gets a result file under
   `output/checks/<check>/`, bound `VERSION` when the brief names a `version`, else `UNIT`.
7. Grade the whole delivery PASS, MINOR or MAJOR with the rubric.
8. Writers write `.cad-work/<unit>/delivery.json` (root `worker-delivery`, fields below) and validate it
   against the brief's schema path → exit 0.
9. Reply in the format below.

Stop and deliver MAJOR with `ready: false` and the reason when: a brief `stop` condition holds; an input or
resource is missing; the task needs a change to a locked or shared file; a required capability, method or tool
is UNKNOWN; a safety-relevant fact is UNKNOWN; the build fails; or, for a producer, a check of its own output is
FAIL or UNKNOWN. A checker or reviewer delivers a FAIL, UNKNOWN or MAJOR finding as its result; that is the
evidence, not a stop.

## Delivery fields

| Field | Value |
|---|---|
| `unit`, `role`, `input_inventory_sha256` | copied from the brief |
| `brief_sha256` | `shasum -a 256` of `brief.yaml` |
| `verdict`, `ready` | the rubric grade; `ready: true` only with PASS |
| `build` | `SUCCEEDED` or `FAILED` with its log file, or `NOT_APPLICABLE` with the reason |
| `outputs[]` | `source` (absolute path under `output/`), `target` (an `expected.outputs` target), `sha256`, `kind` |
| `checks[]` | each expected check: `result_file {path, sha256}`, `result`, `family` |
| `logs[]` | every build and check stdout and stderr file |
| `boundary` | `method: tree digest before first and after last write`, `before_sha256`, `after_sha256`, `violations` (every write outside `writable` you made or saw) |
| `proposals[]` | a change to a locked or shared file: proposal file under `output/proposals/`, `target`, `change`, `reason` |
| `unknowns[]` | typed UNKNOWN facts; consequential ones carry owner and question |
| `limits[]` | what the delivery does not establish (a UNIT result: `UNIT evidence only; integrated VERSION checks rerun`) |

## Roles

| Role | Duty | Skills to LOAD | Writes | Delivers to |
|---|---|---|---|---|
| `cad-concept-designer` | one variant from the brief's use case, requirements, preferences and feasible route: architecture, load paths, interfaces, coarse CAD and GLB when asked, feasibility per hard constraint, unknowns | cad-design, cad-model, cad-select, cad-record | `output/**`, `build.py`, `delivery.json` | `hw/concepts/<variant>/{variant.yaml,architecture.md,geometry/**}` |
| `cad-part-designer` | the brief's PNs: part record, one-kernel model, components, manufacturing state, route, drawing intent at DETAIL, own build, readback and UNIT checks | cad-design, cad-model, cad-select, cad-verify, cad-record; cad-release at DETAIL | `output/**`, `build.py`, `check.py`, `delivery.json` | `hw/parts/<PN>/{part.yaml,model.py,sources/**}` |
| `cad-checker` | independent checks of the brief's version or unit inputs: own programs, results, artifacts, logs, coverage | cad-verify, cad-model, cad-record | `output/**`, `check.py`, `delivery.json` | `hw/checks/<vN>/<check>/<attempt>/**` |
| `cad-mech-dfm-reviewer` | separate mechanical and DFM outcomes for the exact version: premises, requirements, routes, evidence, drawings, BOM | cad-review, cad-verify, cad-select, cad-record | nothing | `hw/checks/<vN>/<check>/<attempt>/result.json` |
| `cad-sourcing-researcher` | the brief's purchased families, stock or capability: retained source bytes, profile proposals, catalogue alternatives with citations | cad-select, cad-record | `output/**`, `delivery.json` | `hw/manufacturing/sources/<sha256>/<name>`, `hw/parts/<PN>/sources/**` |

- Concept designer: NOT compare, rank or select variants.
- Part designer: NOT change an interface, the assembly or another PN; shared changes are proposals.
- Checker: NOT edit design, criteria, cases or sampling; NOT reuse a producer's program as its own method.
- Reviewer: read-only, runs nothing, writes nothing. Its reply carries the review result document as one fenced
  `json` block with `implementation.kind: REVIEW` and `inspected` listing every file read with its sha256 from
  the brief inventory. The lead writes those bytes unchanged and writes the delivery for it.
- Sourcing researcher: profiles are proposals to the shared libraries; NOT purchase, select, approve or write a
  shared record.

## Rules for every role

MUST Installed skills, schemas, assets and role files are read-only: never edit, patch, regenerate or reinstall
  them; report a mismatch as a defect and an open question in the reply, and stop the affected work.
MUST Keep missing evidence UNKNOWN with its reason; never a default, an invented value or a fallback method.
NOT Delegate, spawn agents, approve, ask the user, or write `hw/`, another unit's scratch or any path outside
  `boundary.writable`.
NOT Repair a locked input or report a UNIT PASS as assembly, stage or physical evidence.

## Reply

```text
VERDICT: PASS|MINOR|MAJOR — <reason> — unit <unit> — <subject>
Delivery — <absolute path of delivery.json>
Findings — <finding id, severity, absolute evidence path>   (only if any)
Unknowns — <fact, owner, question>                          (only if any)
Proposals — <target, absolute proposal path>                (only if any)
```

Cap 150 words clean, 300 with findings; structured evidence (files, the reviewer's JSON block) is outside the cap.
Each absolute path on its own line; never reprint inputs, files or code.
