# Agent roles

Owner: cad-flow. The one contract for the five worker roles: duties, inputs, write boundary, outputs and reply. Each native role file links
here and repeats no rule except the install guard. A worker reads this file, its brief, and the skills and rubric its brief lists;
never cad-flow SKILL.md or another lead reference. Grading uses the rubric owned by cad-review:
[the rubric reference](../../cad-review/references/rubric.md).

## Every role

1. Read the brief at the absolute path in your task. Every resource is the literal absolute path or `skill://` URI in `resources`; never
   search the product directory or guess an install path. Your `limits` count from this step: wall seconds from its `date -u` time, model
   turns as your own model replies.
2. LOAD the SKILL.md of each `resources.skills` entry and no other skill, and read the cad-review rubric at `resources.rubric`.
3. Run `shasum -a 256 -c <root>/.cad-work/<unit>/input/SHA256SUMS` once (Linux: `sha256sum -c` of the same file) → every line OK, else
   stop. Read the `input_inventory` files and snapshots from their copies at `.cad-work/<unit>/input/<path>`, and the
   `boundary.read_only` paths; nothing else. A needed file outside them → stop.
4. Writers: take the tree digest with the command of
   [cad-record's Inventory digest](../../cad-record/references/record.md#inventory-digest), no other form, before your first write as
   `boundary.before_sha256`, and again after your last write as `boundary.after_sha256`.
5. Do the role duty below; write only the `boundary.writable` paths of the brief. A temporary file (a path list for a set digest, an edit
   buffer, a probe, a download test) goes under `.cad-work/<unit>/tmp/`, never `/tmp` or the unit root, and every program runs with
   `TMPDIR` set there ([uv commands](#uv-commands)).
6. Writers build and check with their own programs `.cad-work/<unit>/build.py` and `check.py`, locked and run with the
   [uv commands](#uv-commands). Use [cad-model's unit build procedure](../../cad-model/references/exports.md#unit-build) and
   [cad-verify's implementation contract](../../cad-verify/references/results.md#implementation-kind); keep each program's uv lock beside it
   as `build.py.lock` or `check.py.lock`, at the unit root, never under `output/` and never as a delivery output: the lead retains them
   at acceptance. A checker whose `input_inventory` lists an earlier result's `source/check.py` and its lock may start its `check.py` from
   that copy (cad-verify implementation kind, step 2). Each expected check gets its result file under `output/checks/<check>/`, started
   from the brief's check-result template, bound `VERSION` when the brief names a `version`, else `UNIT`; its `source_files` name
   `source/<file>` beside the result's target with each program's sha256, and each command's `stdout` and `stderr` name `stdout.txt` and
   `stderr.txt` beside the result's target, never a scratch path
   ([cad-verify implementation kind](../../cad-verify/references/results.md#implementation-kind) step 1); the run writes them at
   `output/checks/<check>/stdout.txt` and `stderr.txt` and lists both in `logs`. Results come only from the final program run: after the
   last edit to a program or its lock, rerun it fully, rewrite and revalidate every result and artifact it produces, then deliver. Deliver
   after the first full run whose results all validate: edit a program after it only to fix a defect that changes a result, then rerun.
7. Grade the whole delivery PASS, MINOR or MAJOR with the [cad-review rubric](../../cad-review/references/rubric.md).
8. Writers check before delivering: every `outputs` target is literal (a `target_pattern` with `<sha256>` replaced by the sha256 of the
   output's final bytes and `<name>` by its file name); a change to a locked or shared file is only in `proposals`, never in `outputs`;
   every write lies in `boundary.writable`. Then write `.cad-work/<unit>/delivery.json` from the brief's worker-delivery template (root
   [worker-delivery](../../cad-record/schemas/worker-delivery.schema.json), fields below) and validate it with the brief's
   `check-jsonschema` argv against the brief's schema path → exit 0.
9. Reply in the format below.

Stop and deliver MAJOR with `ready: false` and the reason when: a brief `stop` condition holds; a brief `limits` value is reached; an input
or resource is missing; the task needs a change to a locked or shared file; a required capability, method or tool is UNKNOWN; a
safety-relevant fact is UNKNOWN; the build fails; or, for a producer, a check of its own output is FAIL or UNKNOWN. A sourcing subject
fact that no retained source states is no stop ([Roles](#roles)). A checker or reviewer delivers a FAIL, UNKNOWN or MAJOR finding as its
result; that is the evidence, not a stop ([cad-flow worker delivery](worker-delivery.md#accept)).

A host that denies one command has not blocked every command: retry only the exact command forms this contract and your skills name,
each once, and record every denial verbatim (the command and the host's denial text) in `limits`. A required command that stays denied
is a missing tool: stop as above.

## Delivery fields

| Field                                    | Value                                                                                                                                                                  |
| ---------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `unit`, `role`, `input_inventory_sha256` | copied from the [worker brief](worker-brief.md)                                                                                                                        |
| `brief_sha256`                           | `shasum -a 256` of `brief.yaml`                                                                                                                                        |
| `verdict`, `ready`                       | the [cad-review rubric](../../cad-review/references/rubric.md) grade; `ready: true` only with PASS                                                                     |
| `build`                                  | `SUCCEEDED` or `FAILED` with its log file, or `NOT_APPLICABLE` with the reason                                                                                         |
| `outputs[]`                              | `source` (absolute path under `output/`), `target` (an `expected.outputs` target), `sha256`, `kind`                                                                    |
| `checks[]`                               | each expected check's result file and outcome; fields are defined by the [cad-record check-result schema](../../cad-record/schemas/check-result.schema.json)           |
| `logs[]`                                 | every build and check stdout and stderr file ([cad-record worker-delivery schema](../../cad-record/schemas/worker-delivery.schema.json))                               |
| `boundary`                               | tree digests and violations for every write outside the brief's `boundary.writable` ([worker-delivery boundary](../../cad-record/schemas/worker-delivery.schema.json)) |
| `proposals[]`                            | proposal files under `output/proposals/` for locked/shared `hw/` changes, with target, change and reason                                                               |
| `unknowns[]`                             | typed UNKNOWN facts; consequential ones carry owner and question                                                                                                       |
| `limits[]`                               | what the delivery does not establish (a UNIT result: `UNIT evidence only; integrated VERSION checks rerun`)                                                            |

## Roles

| Role                      | Duty                                                                                                                                                                                                                                                                                                                            | Skills a brief may list                                                                     | Writes                                                                                                                 | Delivers to                                                                 |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| `cad-concept-designer`    | one variant from the brief's use case, requirements, preferences and feasible route: architecture, load paths, interfaces, coarse CAD and GLB when asked, feasibility per hard constraint, unknowns, own build and geometry readback                                                                                            | cad-design, cad-model, cad-select, cad-verify, cad-record                                   | `output/**`, `build.py`, `build.py.lock`, `check.py`, `check.py.lock`, `tmp/**`, `__pycache__/**`, `delivery.json`       | `hw/concepts/<variant>/{variant.yaml,architecture.md,geometry/**}`          |
| `cad-part-designer`       | the brief's part numbers and any required assembly drawing: part record, one-kernel model, components, manufacturing state, route, part drawing intent at DETAIL, own build, readback and UNIT checks; draw each briefed subject per [cad-release's drawing procedure](../../cad-release/references/drawings.md#draft-drawings) | cad-design, cad-model, cad-select, cad-verify, cad-record; cad-release at DETAIL and FREEZE | `output/**`, `build.py`, `build.py.lock`, `check.py`, `check.py.lock`, `tmp/**`, `__pycache__/**`, `delivery.json`       | `hw/parts/<PN>/{part.yaml,model.py,sources/**}` and briefed drawing bundles |
| `cad-checker`             | independent checks of the brief's version or unit inputs: own programs, results, artifacts, logs, coverage                                                                                                                                                                                                                      | cad-verify, cad-model, cad-select, cad-design, cad-release, cad-record                      | `output/**`, `check.py`, `check.py.lock`, `tmp/**`, `__pycache__/**`, `delivery.json`                                    | `hw/checks/<vN>/<check>/<attempt>/**`                                       |
| `cad-mech-dfm-reviewer`   | separate mechanical and DFM outcomes for the exact version: premises, requirements, routes, evidence, drawings, BOM                                                                                                                                                                                                             | cad-review, cad-verify, cad-select, cad-record                                              | nothing                                                                                                                | `hw/checks/<vN>/<check>/<attempt>/result.json`                              |
| `cad-sourcing-researcher` | the brief's purchased families, stock or capability: retained source bytes, profile proposals, catalogue alternatives with citations                                                                                                                                                                                            | cad-select, cad-record                                                                      | `output/**`, `extract.py`, `extract.py.lock`, `tmp/**`, `__pycache__/**`, `delivery.json`                              | `hw/manufacturing/sources/<sha256>/<name>`, `hw/parts/<PN>/sources/**`      |

The brief lists only the skills that own its methods ([worker-brief](worker-brief.md#compose)); a role loads no other. Every writer
also writes the shared uv cache, through uv only ([uv commands](#uv-commands)).

- Concept designer: NOT compare, rank or select variants ([cad-design](../../cad-design/references/concepts.md#compare-variants)).
- Part designer: NOT change an interface, the assembly or another PN; shared changes are proposals.
- Reviewer: read-only: runs no build or check program and writes nothing
  ([cad-review rubric](../../cad-review/references/rubric.md)). On Codex, which has no file-read tool without the shell, it reads
  through shell read commands and stays read-only by instruction; the lead's tree digest before dispatch and after return
  ([worker delivery — Read-only reviewer](worker-delivery.md#read-only-reviewer)) is the check. Its reply carries the review result
  document as one fenced `json` block with `implementation.kind: REVIEW` and `inspected` listing every file read with its sha256 from
  the brief inventory. The lead writes those bytes unchanged and writes the delivery for it.
- Sourcing researcher: profiles are proposals to the shared libraries ([cad-select materials](../../cad-select/references/materials.md));
  NOT purchase, select, approve or write a shared record. Retain each source's original bytes with one of these two commands, never a
  web-fetch text conversion; the second adds `--create-dirs` when the directory of `<path>` does not exist yet. The retained bytes are
  the body curl wrote, hashed with `shasum -a 256` (or `sha256sum`), and each is delivered to a brief pattern with `<sha256>` its hash:

  ```sh
  curl -fsSL -o <root>/.cad-work/<unit>/output/<path> --compressed --remove-on-error --connect-timeout 10 --max-time 60 --max-filesize 64M --retry 1 --retry-max-time 20 <url>
  curl -fsSL --create-dirs -o <root>/.cad-work/<unit>/output/<path> --compressed --remove-on-error --connect-timeout 10 --max-time 60 --max-filesize 64M --retry 1 --retry-max-time 20 <url>
  ```

  To read a retained PDF's text, write the PEP 723 program `.cad-work/<unit>/extract.py`, lock it beside itself as `extract.py.lock`
  and run it with the [uv commands](#uv-commands); it reads the retained bytes unchanged and writes the text to its own file under
  `output/`. The text is derived evidence, never a substitute for the original bytes, which stay retained and are what a fact cites;
  deliver the text as its own source, with `<sha256>` its own hash. `extract.py` and its lock stay in the unit directory and are never
  delivery outputs.

  Budget per unit: at most 8 retained sources, 16 fetch attempts and 3 attempts per host; a host that answered 403 or 429 or failed TLS
  is not fetched again. An attempt is one curl run that reached the host; a run the host denied, or that failed on the sandbox's DNS
  before connecting, and its one re-run after the host's escalation count as one attempt. Count before each fetch: the attempt that
  would exceed a budget is not run. Discover only through the host's search tool or the brief's `seed_urls`; never scan an archive CDX
  index or a store's product JSON.

  A subject fact that no retained source states, within the budget, is the research result for it: a typed UNKNOWN with owner and
  question in `unknowns`, never a stop and never a finding by itself; the delivery grades on the rest
  ([cad-review rubric](../../cad-review/references/rubric.md)). The attempt log (URL → outcome) goes in `limits`.

## uv commands

Every uv command of a unit or of the lead uses the one shared cache, runs offline and never downloads Python. `<cache>` is
`<root>/.cad-work/lead/.uv-cache`, warmed by the lead ([preflight Warm the uv cache](preflight.md#warm-the-uv-cache)); `<tmp>` is
`<root>/.cad-work/<unit>/tmp` for a unit and `<root>/.cad-work/lead/tmp` for the lead; `<program>` is the absolute path of the PEP 723
script:

```sh
TMPDIR=<tmp> uv lock --cache-dir <cache> --no-python-downloads --offline --script <program>
TMPDIR=<tmp> uv run --cache-dir <cache> --no-python-downloads --offline --locked --script <program> <arguments>
```

- The interpreter is the Python 3.12 preflight found ([preflight Probe tools](preflight.md#probe-tools)); none found is a missing tool.
- Every program except the sourcing `extract.py` declares exactly one warm set as its PEP 723 header, `requires-python = "==3.12.*"`
  and these `dependencies`; any other header cannot lock offline, which is a missing tool:

  | Warm set                | `dependencies`                                                                                                                                                                                   |
  | ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
  | `yaml`                  | the [pinned PyYAML](../../cad-record/references/validation.md#yaml-loader)                                                                                                                       |
  | `build123d`, `cadquery` | that kernel's pins in [cad-model Kernels and environments](../../cad-model/references/model-contract.md#kernels-and-environments) and the pinned PyYAML                                          |

- A program that parses record YAML does it with the PyYAML of its warm set
  ([cad-record YAML loader](../../cad-record/references/validation.md#yaml-loader)). The sourcing researcher's `extract.py` runs both
  commands without `--offline`, on the network its fetches use; the host's escalation for them is no fetch attempt.
- Lock once, then only run: `uv run --locked` reuses the resolved environment, and a program is relocked only after a header change
  ([cad-model Kernels and environments](../../cad-model/references/model-contract.md#kernels-and-environments)). The lead's `checks.py`
  follows the same rule on every check run.
- `<cache>` lies under the project root, inside every host's workspace and sandbox, so a unit reads it without network. uv writes there
  only its own content-addressed entries and supports concurrent readers and writers. `<cache>`, a unit's `tmp/` and any `__pycache__/`
  lie outside every digest and are never an output or evidence ([worker-delivery Accept](worker-delivery.md#accept) step 8).
- Schema validation and a YAML load outside a program run only the brief's `resources.tools` argv, prefixed `TMPDIR=<tmp>`: binaries
  preflight installed, executed read-only, never resolved through `uvx`.

## Rules for every role

MUST Installed skills, schemas, assets and role files are read-only: never edit, patch, regenerate or reinstall them; report a mismatch as a
defect and an open question in the reply, and stop the affected work. MUST Keep missing evidence UNKNOWN with its reason, following the
[cad-record fact contract](../../cad-record/references/record.md#facts); never a default, an invented value or a fallback method. NOT
Delegate, spawn agents, approve, ask the user, or write `hw/`, another unit's scratch or any path outside `boundary.writable`
([worker-brief boundary](worker-brief.md#compose)). NOT Repair a locked input or report a UNIT PASS as assembly, stage or physical evidence.

## Reply

```text
VERDICT: PASS|MINOR|MAJOR — <reason> — unit <unit> — <subject>
Delivery — <absolute path of delivery.json>
Findings — <finding id, severity, absolute evidence path>   (only if any)
Unknowns — <fact, owner, question>                          (only if any)
Proposals — <target, absolute proposal path>                (only if any)
```

Cap 150 words clean, 300 with findings; structured evidence (files, the reviewer's JSON block) is outside the cap
([cad-review rubric](../../cad-review/references/rubric.md)). Each absolute path on its own line; never reprint inputs, files or code.
