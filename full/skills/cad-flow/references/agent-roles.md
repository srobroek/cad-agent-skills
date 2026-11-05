# Agent roles

Owner: cad-flow. The one contract for the five worker roles: duties, inputs, write boundary, outputs and reply. Each native role file links
here and repeats no rule except the install guard. A worker reads only its [read set](#read-set): this file, its brief and inputs, and
the skills, rubric and resources its brief lists; never cad-flow SKILL.md or another lead reference. Grading uses the rubric owned by
cad-review: [the rubric reference](../../cad-review/references/rubric.md).

## Every role

1. Read the brief at the absolute path in your task, then, with a shell, run `date -u +%Y-%m-%dT%H:%M:%SZ`: your [limits](#limits) count
   from the time it prints. Every resource is the literal absolute path or `skill://` URI in `resources`; never search the product
   directory or guess an install path.
2. LOAD the SKILL.md of each `resources.skills` entry and no other skill, and read the cad-review rubric at `resources.rubric`; read the
   other files of those skills as their SKILL.md and your methods direct.
3. Except the reviewer, run `shasum -a 256 -c <root>/.cad-work/<unit>/input/SHA256SUMS` once (Linux: `sha256sum -c` of the same file)
   → every line OK, else stop. Read the `input_inventory` files and snapshots from their copies at `.cad-work/<unit>/input/<path>`; a
   needed file outside your [read set](#read-set) → stop. An earlier result's sha256 for `input/lead/attempts.txt` is confirmed against
   your copy cut after that result's attempt line ([worker brief Compose](worker-brief.md#compose) step 2). Every file you write cites
   an input by its origin, the `input_inventory.files` path (`hw/…`) with the brief's sha256 for it, never by its `input/` copy; a
   snapshot has no `hw/` origin and is never a path field, because the brief binds it through `brief_sha256`
   ([cad-record validation — Semantic check](../../cad-record/references/validation.md#semantic-check) item 12).
4. Writers: take the tree digest with the command of
   [cad-record's Inventory digest](../../cad-record/references/record.md#inventory-digest), no other form, before your first write as
   `boundary.before_sha256`, and again after your last write as `boundary.after_sha256`.
5. Do the role duty below; write only the `boundary.writable` paths of the brief. A temporary file (a path list for a set digest, an edit
   buffer, a probe, a download test) goes under `.cad-work/<unit>/tmp/`, never `/tmp` or the unit root, and every program runs with
   `env TMPDIR=<tmp>` set there ([uv commands](#uv-commands)).
6. Writers build and check with their own programs `.cad-work/<unit>/build.py` and `check.py`, locked and run with the
   [uv commands](#uv-commands). Use [cad-model's unit build procedure](../../cad-model/references/exports.md#unit-build) and
   [cad-verify's implementation contract](../../cad-verify/references/results.md#implementation-kind); keep each program's uv lock beside it
   as `build.py.lock` or `check.py.lock`, at the unit root, never under `output/` and never as a delivery output: the lead retains them
   at acceptance. A checker whose `input_inventory` lists an earlier result's `source/check.py` and its lock may start its `check.py` from
   that copy (cad-verify implementation kind, step 2):
   `cp <root>/.cad-work/<unit>/input/hw/checks/<vN>/<check>/<attempt>/source/check.py <root>/.cad-work/<unit>/check.py`, and the same
   for `check.py.lock`. A lite geometry checker ([stages Build mode](stages.md#build-mode)) also writes, locks and runs its own `build.py`
   before `check.py` when its brief's `boundary.writable` lists `build.py` and `build.py.lock`
   ([cad-model exports — geometry.build](../../cad-model/references/exports.md#geometrybuild), Lite), and its `check.py` alone runs the
   brief's `freecad` tool, as a subprocess on the helper `tmp/<check>/freecad-readback.FCScript` it writes
   ([cad-model exports — FreeCAD readback](../../cad-model/references/exports.md#freecad-readback)); never run `freecad` from the shell,
   and never edit the helper after its run. Each expected check gets its result file under `output/checks/<check>/`, started from the
   brief's check-result template, bound `VERSION` when the brief names a `version`, else `UNIT`; its `source_files` name
   `source/<file>` beside the result's target with each program's sha256, and each command's `stdout` and `stderr` name `stdout.txt` and
   `stderr.txt` beside the result's target (a lite checker's `build.py` run: the `artifacts/` logs its method names), never a scratch path
   ([cad-verify implementation kind](../../cad-verify/references/results.md#implementation-kind) step 1); the run writes them at
   `output/checks/<check>/stdout.txt` and `stderr.txt` and lists both in `logs`. Every `artifacts[]` entry of a result names its brief
   target (a literal one, or the brief's `artifacts/<name>` pattern with `<name>` its file name), is written under
   `output/checks/<check>/artifacts/` and is delivered as an output. Results come only from the final program run: after the
   last edit to a program or its lock, rerun it fully, rewrite and revalidate every result and artifact it produces, then deliver. Deliver
   after the first full run whose results all validate: edit a program after it only to fix a defect that changes a result, then rerun.
7. Grade the whole delivery PASS, MINOR or MAJOR with the [cad-review rubric](../../cad-review/references/rubric.md).
8. Writers check before delivering: every `outputs` target is literal (a `target_pattern` with `<sha256>` replaced by the sha256 of the
   output's final bytes and `<name>` by its file name); a change to a locked or shared file is only in `proposals`, never in `outputs`;
   every write lies in `boundary.writable`. Then write `.cad-work/<unit>/delivery.json` from the brief's worker-delivery template (root
   [worker-delivery](../../cad-record/schemas/worker-delivery.schema.json), fields below) and validate it with this one command, never
   another parser, its argv and schema path copied from the brief → exit 0:
   `env TMPDIR=<tmp> <check-jsonschema argv> --schemafile <worker-delivery schema path> <root>/.cad-work/<unit>/delivery.json`.
   Every sha256 you record is pasted from `shasum -a 256` output,
   never typed; confirm them last: write one `<sha256>  <absolute path>` line per file the delivery hashes (`brief.yaml`, each output,
   result file, log and proposal) to `.cad-work/<unit>/tmp/delivery.sha256` and run
   `shasum -a 256 -c <root>/.cad-work/<unit>/tmp/delivery.sha256` (Linux: `sha256sum -c`) → every line OK; a failed line is corrected
   in `delivery.json` from fresh `shasum -a 256` output, and this step repeats.
9. Reply in the format below.

Stop and deliver MAJOR with `ready: false` and the reason when: a brief `stop` condition holds; an input or resource is missing; the task
needs a change to a locked or shared file; a required capability, method or tool is UNKNOWN; a safety-relevant fact is UNKNOWN; the build
fails; or, for a producer, a check of its own output is FAIL or UNKNOWN. A reached limit follows [Limits](#limits). For a sourcing
researcher, a subject fact that no retained source states and a failed search tool are no stop ([Roles](#roles)). A checker or reviewer
delivers a FAIL, UNKNOWN or MAJOR finding as its result; that is the evidence, not a stop
([cad-flow worker delivery](worker-delivery.md#accept)).

Run each command alone, never chained with `;`, `&&`, `||` or `|`: the one pipeline is the tree digest of step 4, and a program run's
`> <stdout> 2> <stderr>` redirect is part of its form. The shell runs only the command forms this contract and your skills name. List,
read and search files with the host's file tools (on Codex, which has none, with read-only shell commands), never a shell `cat`, `head`,
`wc`, `grep`, `diff` or `file`, and no `find`, `sed` or `sort` outside the digest commands; move or rename nothing. The one shell
listing is `ls -la <root>/.cad-work/<unit>/<dir>` of your own unit (`<dir>` empty or one of its directories). The one directory command
is `mkdir -p <root>/.cad-work/<unit>/<dir>`, one absolute directory per call; a write tool and curl's `--create-dirs` make their own
parents. A host that denies one command has not blocked every command: retry only the exact command forms this contract and your
skills name, each once, and record every denial verbatim (the command and the host's denial text) in `limits`. A required command
that stays denied is a missing tool: stop as above.

## Read set

You read only:

- your unit directory `<root>/.cad-work/<unit>/`: the brief, the `input/` copies and the files you write;
- the installed files of each `resources.skills` skill (every file under the directory of its SKILL.md: references, schemas, assets)
  and each other `resources` path: role reference, rubric, schemas, templates, `layout` and `policy`. A search (the host's grep or glob)
  rooted at a listed skill's directory reads that skill; one rooted above it, at the skills directory or the project, does not;
- the `resources.tools` binaries, which you execute and never list.

Never another `.cad-work/` directory (`.cad-work/lead/` with its `preflight.json`, `units.yaml`, `checks.py` and `validator/`, a
`stage-v*` directory, another unit's), a live `hw/` file, a `boundary.locked` one included (read its input copy), an installed skill the
brief does not list, or the product tree by listing or search. The tree digest of step 4 hashes the tree without your reading its files,
and uv alone uses the shared cache. Your templates are your only examples of a file's shape, never another unit's files; a path, version
or header the brief does not give is in your skills ([uv commands](#uv-commands)), never a reason to look further.

Reading a file's contents outside the read set is a boundary defect: record each such path in `limits`, and the lead rejects the
delivery ([worker delivery Accept](worker-delivery.md#accept) step 8). A names-only listing outside it (a directory listing, a glob or
search that returns names) reads no contents: record it in `limits` as an INFO observation; it changes no grade and rejects nothing.

## Limits

`limits` count from the time step 1 printed: `wall_seconds` of wall time, and `model_turns` of your model replies, a reply that calls
tools counting once. Before each program run and each fetch, and at least every 20 turns, run `date -u +%Y-%m-%dT%H:%M:%SZ` and compare
both with the brief. Once 90% of either is spent, start no new work and deliver what you have; the lead stops a unit still running at
its wall limit ([delegation Lost and stalled units](delegation.md#lost-and-stalled-units)).

- A checker runs its program once more, recording every obligation it has not established UNKNOWN `TIMEOUT`
  ([cad-verify results Outcomes](../../cad-verify/references/results.md#outcomes)); the result is UNKNOWN and the grade MAJOR.
- The reviewer records every obligation it has not assessed UNKNOWN `TIMEOUT` and grades MAJOR; without a shell it counts turns only.
- A sourcing researcher records every subject fact no retained source states yet as a typed UNKNOWN whose reason names the limit.
- A designer delivers MAJOR with `ready: false`.

The delivery's `limits` names the limit reached, with the time and turns spent. A limit never raises a grade: unfinished work is
UNKNOWN, never PASS.

## Delivery fields

| Field                                    | Value                                                                                                                                                                  |
| ---------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `unit`, `role`, `input_inventory_sha256` | copied from the [worker brief](worker-brief.md)                                                                                                                        |
| `brief_sha256`                           | the sha256 `shasum -a 256 <root>/.cad-work/<unit>/brief.yaml` prints, pasted from its output ([Every role](#every-role) step 8)                                    |
| `verdict`, `ready`                       | the [cad-review rubric](../../cad-review/references/rubric.md) grade; `ready: true` only with PASS                                                                     |
| `build`                                  | `SUCCEEDED` or `FAILED` with its log file, or `NOT_APPLICABLE` with the reason                                                                                         |
| `outputs[]`                              | `source` (absolute path under `output/`), `target` (an `expected.outputs` target), `sha256`, `kind`                                                                    |
| `checks[]`                               | each expected check's result file, with `result` and `family` equal to that file's; fields are defined by the [cad-record check-result schema](../../cad-record/schemas/check-result.schema.json) |
| `logs[]`                                 | every build and check stdout and stderr file ([cad-record worker-delivery schema](../../cad-record/schemas/worker-delivery.schema.json))                               |
| `boundary`                               | tree digests and violations for every write outside the brief's `boundary.writable` ([worker-delivery boundary](../../cad-record/schemas/worker-delivery.schema.json)) |
| `proposals[]`                            | proposal files under `output/proposals/` for locked/shared `hw/` changes, with target, change and reason                                                               |
| `unknowns[]`                             | typed UNKNOWN facts; consequential ones carry owner and question                                                                                                       |
| `limits[]`                               | what the delivery does not establish (a UNIT result: `UNIT evidence only; integrated VERSION checks rerun`), a reached [limit](#limits) and every denied command |

## Roles

| Role                      | Duty                                                                                                                                                                                                                                                                                                                            | Skills a brief may list                                                                     | Writes                                                                                                                 | Delivers to                                                                 |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| `cad-concept-designer`    | one variant from the brief's use case, requirements, preferences and feasible route: architecture, load paths, interfaces, coarse CAD and GLB when asked, feasibility per hard constraint, unknowns, own build and geometry readback                                                                                            | cad-design, cad-model, cad-select, cad-verify, cad-record                                   | `output/**`, `build.py`, `build.py.lock`, `check.py`, `check.py.lock`, `tmp/**`, `__pycache__/**`, `delivery.json`       | `hw/concepts/<variant>/{variant.yaml,architecture.md,geometry/**}`          |
| `cad-part-designer`       | the brief's part numbers and any required assembly drawing: part record, one-kernel model, components, manufacturing state, route, part drawing intent at DETAIL, own build, readback and UNIT checks; draw each briefed subject per [cad-release's drawing procedure](../../cad-release/references/drawings.md#draft-drawings); in a lite build no manufacturing state or route ([stages Build mode](stages.md#build-mode)) | cad-design, cad-model, cad-select, cad-verify, cad-record; cad-release at DETAIL and FREEZE | `output/**`, `build.py`, `build.py.lock`, `check.py`, `check.py.lock`, `tmp/**`, `__pycache__/**`, `delivery.json`       | `hw/parts/<PN>/{part.yaml,model.py,sources/**}` and briefed drawing bundles |
| `cad-checker`             | independent checks of the brief's version or unit inputs: own programs, results, artifacts, logs, coverage; in a lite build also the LAYOUT intent inspection, a REVIEW result ([cad-verify families](../../cad-verify/references/families.md#lite-intent-inspection)) | cad-verify, cad-model, cad-select, cad-design, cad-release, cad-record                      | `output/**`, `check.py`, `check.py.lock`, `tmp/**`, `__pycache__/**`, `delivery.json`; a lite geometry checker also `build.py`, `build.py.lock` when its brief lists them | `hw/checks/<vN>/<check>/<attempt>/**`                                       |
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
  NOT purchase, select, approve or write a shared record. Retain only bytes the vendor's own origin serves: a host of the manufacturer's
  or supplier's own domain, or a file host the vendor's own page links for that document (for a standard, its publisher). A mirror,
  reseller, marketplace, forum, archive, university or other third-party copy is never retained, a redirect that leaves the vendor's
  origin is such a copy, and a fact only such a copy states is a typed UNKNOWN. Retain each source's original bytes with one of these
  two commands, never a web-fetch text conversion; the second adds `--create-dirs` when the directory of `<path>` does not exist yet.
  The retained bytes are the body curl wrote, hashed with `shasum -a 256` (or `sha256sum`), and each is delivered to a brief pattern
  with `<sha256>` its hash:

  ```sh
  curl -fsSL -o <root>/.cad-work/<unit>/output/<path> --compressed --remove-on-error --connect-timeout 10 --max-time 60 --max-filesize 64M --retry 1 --retry-max-time 20 <url>
  curl -fsSL --create-dirs -o <root>/.cad-work/<unit>/output/<path> --compressed --remove-on-error --connect-timeout 10 --max-time 60 --max-filesize 64M --retry 1 --retry-max-time 20 <url>
  ```

  To read a retained PDF's text, write the PEP 723 program `.cad-work/<unit>/extract.py`, lock it beside itself as `extract.py.lock`
  and run it with the [uv commands](#uv-commands); it reads the retained bytes unchanged and writes the text to its own file under
  `output/`. The text is derived evidence, never a substitute for the original bytes, which stay retained and are what a fact cites;
  deliver the text as its own source, with `<sha256>` its own hash. `extract.py` and its lock stay in the unit directory and are never
  delivery outputs. `extract.py` is your one program and the only program that runs online; run no other program, probe or YAML load:
  your YAML proposals pass `check-jsonschema` alone, and the lead loads them when it applies them.

  Budget per unit: at most 8 retained sources, 16 fetch attempts and 3 attempts per host; a host that answered 403 or 429 or failed TLS
  is not fetched again. An attempt is one curl run that reached the host; a run the host denied, or that failed on the sandbox's DNS
  before connecting, and its one re-run after the host's escalation count as one attempt. Count before each fetch: the attempt that
  would exceed a budget is not run. Discover only through the host's search tool, the brief's `seed_urls` and the links of pages
  fetched from a vendor's own origin; never scan an archive CDX index or a store's product JSON. A search tool that is absent, fails or
  is cancelled is no missing tool and no stop: record its error text in `limits` and discover on from the seeds.

  A subject fact that no retained source states, within the budget and your [limits](#limits), is the research result for it: a typed
  UNKNOWN with owner and question in `unknowns`, never a stop and never a finding by itself; the delivery grades on the rest
  ([cad-review rubric](../../cad-review/references/rubric.md)). The attempt log (URL → outcome) goes in `limits`.

## uv commands

Every uv command of a unit or of the lead uses the one shared cache, runs offline (the sourcing `extract.py` alone excepted) and never
downloads Python. `<cache>` is `<root>/.cad-work/lead/.uv-cache`, warmed by the lead
([preflight Warm the uv cache](preflight.md#warm-the-uv-cache)); `<tmp>` is `<root>/.cad-work/<unit>/tmp` for a unit and
`<root>/.cad-work/lead/tmp` for the lead; `<program>` is the absolute path of the PEP 723 script. Every program-running command starts
with the `env` utility, never a bare `TMPDIR=<tmp>` prefix, which some hosts strip before they match a command; a program run's
`> <stdout> 2> <stderr>` redirect follows `<arguments>`:

```sh
env TMPDIR=<tmp> uv lock --cache-dir <cache> --no-python-downloads --offline --script <program>
env TMPDIR=<tmp> uv run --cache-dir <cache> --no-python-downloads --offline --locked --script <program> <arguments>
```

- uv takes the Python 3.12 that preflight found ([preflight Probe tools](preflight.md#probe-tools)) from the header's `requires-python`;
  never look up an interpreter, uv or tool path. A uv error that no 3.12 interpreter exists is a missing tool.
- Every program except the sourcing `extract.py` declares exactly one warm set as its PEP 723 header, `requires-python = "==3.12.*"`
  and these `dependencies`; any other header cannot lock offline, which is a missing tool:

  | Warm set                | `dependencies`                                                                                                                                                                                   |
  | ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
  | `yaml`                  | the [pinned PyYAML](../../cad-record/references/validation.md#yaml-loader)                                                                                                                       |
  | `build123d`, `cadquery` | that kernel's pins in [cad-model Kernels and environments](../../cad-model/references/model-contract.md#kernels-and-environments) and the pinned PyYAML                                          |

  A `yaml` header, `<pin>` being the pinned PyYAML version:

  ```python
  # /// script
  # requires-python = "==3.12.*"
  # dependencies = ["pyyaml==<pin>"]
  # ///
  ```

- A program that parses record YAML does it with the PyYAML of its warm set
  ([cad-record YAML loader](../../cad-record/references/validation.md#yaml-loader)). The sourcing researcher's `extract.py`, the only
  program that runs online, runs both commands without `--offline`, on the network its fetches use; the host's escalation for them is
  no fetch attempt.
- Lock once, then only run: `uv run --locked` reuses the resolved environment, and a program is relocked only after a header change
  ([cad-model Kernels and environments](../../cad-model/references/model-contract.md#kernels-and-environments)). The lead's `checks.py`
  follows the same rule on every check run.
- `<cache>` lies under the project root, inside every host's workspace and sandbox, so a unit reads it without network. uv writes there
  only its own content-addressed entries and supports concurrent readers and writers. `<cache>`, a unit's `tmp/` and any `__pycache__/`
  lie outside every digest and are never an output or evidence ([worker-delivery Accept](worker-delivery.md#accept) step 8), except the
  lite FreeCAD helper `tmp/<check>/freecad-readback.FCScript`, which its result retains as a source file ([Every role](#every-role) step 6).
- Schema validation and a YAML load outside a program run only the brief's `resources.tools` argv, prefixed `env TMPDIR=<tmp>`:
  binaries preflight installed, executed read-only, never resolved through `uvx`.

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
