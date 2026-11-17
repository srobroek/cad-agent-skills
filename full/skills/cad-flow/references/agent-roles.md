# Agent roles

Owner: cad-flow. The one contract for the five worker roles: duties, inputs, write boundary, outputs and reply. Each native role file links
here and repeats no rule except the install guard. A worker reads only its [read set](#read-set): this file, its brief, inputs and
resources, and any installed file of this plugin. It follows only this file, its brief and the skills its brief lists, never cad-flow
SKILL.md or another lead reference. A link below tagged `lead-side` leads to such a reference: you need not read it, and the one rule
you act on is stated beside it. Grading uses the rubric owned by
cad-review: [the rubric reference](../../cad-review/references/rubric.md).

## Every role

1. Read the brief at the absolute path in your task, then, with a shell, run `date -u +%Y-%m-%dT%H:%M:%SZ`: your [limits](#limits) count
   from the time it prints. Every resource is the literal absolute path or `skill://` URI in `resources`; never search the product
   tree outside the installed skills directory ([read set](#read-set)) or guess an install path.
2. LOAD the SKILL.md of each `resources.skills` entry and no other skill, and read the cad-review rubric at `resources.rubric`; read the
   other files of those skills as their SKILL.md and your methods direct. Another installed file you read is reference only: it adds no
   method, rule or step to those this file, your brief and the loaded skills give.
3. Except the reviewer, run `shasum -a 256 -c <root>/.cad-work/<unit>/input/SHA256SUMS` once (Linux: `sha256sum -c` of the same file) →
   every line OK, else stop. Read the `input_inventory` files and snapshots from their copies at `.cad-work/<unit>/input/<path>`; a needed
   file outside your [read set](#read-set) → stop. An earlier result's sha256 for `input/lead/attempts.txt` is confirmed against your copy
   cut after that result's attempt line ([lead-side: worker brief Compose](worker-brief.md#compose) step 2). Every file you write cites an
   input by its origin, the `input_inventory.files` path (`hw/…`) with the brief's sha256 for it, never by its `input/` copy. A path field
   is a published `hw/` path; an input snapshot or installed template is named only in a message and bound by the brief's hashes
   ([cad-record validation — Path fields](../../cad-record/references/validation.md#path-fields); the lead rejects any other value:
   [lead-side: worker delivery Accept](worker-delivery.md#accept) step 6).
4. Writers: take the tree digest with the command of
   [cad-record's Inventory digest](../../cad-record/references/record.md#inventory-digest), no other form, before your first write as
   `boundary.before_sha256`, and again after your last write as `boundary.after_sha256`.
5. Do the duty of your [role file](#roles); write only the `boundary.writable` paths of the brief. A temporary file (a path list for a set
   digest, an edit buffer, a probe, a download test) goes under `.cad-work/<unit>/tmp/`, never `/tmp` or the unit root, and every program
   runs with `env TMPDIR=<tmp>` set there ([uv commands](#uv-commands)).
6. Writers build and check with their own programs `.cad-work/<unit>/build.py` and `check.py`, locked and run with the [uv
   commands](#uv-commands). Use [cad-model's unit build procedure](../../cad-model/references/exports.md#unit-build) and [cad-verify's
   implementation contract](../../cad-verify/references/results.md#implementation-kind); keep each program's uv lock beside it as
   `build.py.lock` or `check.py.lock`, at the unit root, never under `output/` and never as a delivery output: the lead retains them at
   acceptance. A checker's extra program rules are in [its role file](agent-roles/cad-checker.md#checker-programs). Each expected check
   gets its result file under `output/checks/<check>/`, started from the brief's check-result template, bound `VERSION` when the brief
   names a `version`, else `UNIT`; its `source_files` name `source/<file>` beside the result's target with each program's sha256, and each
   command's `stdout` and `stderr` name `stdout.txt` and `stderr.txt` beside the result's target (a lite checker's `build.py` run: the
   `artifacts/` logs its method names), never a scratch path ([cad-verify implementation
   kind](../../cad-verify/references/results.md#implementation-kind) step 1); the run writes them at `output/checks/<check>/stdout.txt`
   and `stderr.txt` and lists both in `logs`. Every `artifacts[]` entry of a result names its brief target (a literal one, or the brief's
   `artifacts/<name>` pattern with `<name>` its file name), is written under `output/checks/<check>/artifacts/` and is delivered as an
   output. Results come only from the final program run: after the last edit to a program or its lock, rerun it fully, rewrite and
   revalidate every result and artifact it produces, then deliver. Deliver after the first full run whose results all validate: edit a
   program after it only to fix a defect that changes a result, then rerun.
7. Grade the whole delivery PASS, MINOR or MAJOR with the [cad-review rubric](../../cad-review/references/rubric.md).
8. Writers check before delivering: every `outputs` target is literal (a `target_pattern` with `<sha256>` replaced by the sha256 of the
   output's final bytes and `<name>` by its file name); a change to a locked or shared file is only in `proposals`, never in `outputs`;
   every write lies in `boundary.writable`. Scan your paths: read every path value of each result, each bundle manifest and each cited
   artifact JSON under `output/**`, with the host's file tools. Each value is either an `hw/` path in the brief's
   `input_inventory.files`, or an output target of the brief or a file of the result's own bundle (`source/`, `stdout.txt`,
   `stderr.txt`, `artifacts/`), with no `#fragment` and no absolute path. A `record.semantic` result and its artifacts also
   take the paths of their own `artifacts/seal.sha256` and the missing-root references that
   [Accept](worker-delivery.md#accept) step 6 permits there. Fix each other value (step 3). A field that [cad-record
   validation: Path fields](../../cad-record/references/validation.md#path-fields) lists as not a path field is explained in `limits`
   instead. The misses seen are `hw/…#fragment` in a `locus` (name the anchor in prose and cite the bare file), absolute paths, a live
   `hw/` path outside the inventory, and a package-relative `path` that is no `hw/` path. The lead's [lead-side:
   Accept](worker-delivery.md#accept) scan stays the authority. Then write `.cad-work/<unit>/delivery.json` from the brief's
   worker-delivery template (root [worker-delivery](../../cad-record/schemas/worker-delivery.schema.json), fields below) and validate
   it with this one command, never another parser, its argv and schema path copied from the brief → exit 0:
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
researcher, a subject fact that no retained source states and a failed search tool are no stop ([sourcing
researcher](agent-roles/cad-sourcing-researcher.md#rules)). A checker or reviewer delivers a FAIL, UNKNOWN or MAJOR finding as its result;
that is the evidence, not a stop ([lead-side: cad-flow worker delivery](worker-delivery.md#accept)).

A unit whose `task` starts with `REPAIR of` is a repair unit (read only [delegation Repair units](delegation.md#repair-units)): fix only
the defects its task lists, start from the `rejected/` copies of its inputs (a program from its copy, as in step 6), rewrite and
revalidate what the fix changes, rebind your results and keep every other output as that section says, and deliver with the same checks
(no path field cites a `rejected/` copy: Path fields name `hw/` origins and targets only); any other defect you find goes in `limits`,
never fixed. Its limits are the brief's, shorter than a full unit's.

Run each command alone, never chained with `;`, `&&`, `||` or `|`: the one pipeline is the tree digest of step 4, and a program run's
`> <stdout> 2> <stderr>` redirect is part of its form. The shell runs only the command forms this contract and your skills name. List,
read and search files with the host's file tools (on Codex, which has none, with read-only shell commands), never a shell `cat`, `head`,
`wc`, `grep`, `diff` or `file`, and no `find`, `sed` or `sort` outside the digest commands; move or rename nothing. The one shell
listing is `ls -la <root>/.cad-work/<unit>/<dir>` of your own unit (`<dir>` empty or one of its directories). The one directory command
is `mkdir -p <root>/.cad-work/<unit>/<dir>`, one absolute directory per call; a write tool and curl's `--create-dirs` make their own
parents. A host that denies one command has not blocked every command: retry only the exact command forms this contract and your
skills name, each once, and record every denial verbatim (the command and the host's denial text) in `limits`. A required command
that stays denied is a missing tool: stop as above.

Shell use is graded by the lead ([lead-side: worker delivery Accept](worker-delivery.md#accept) step 11): a shell `cat`, `grep` or
`head` on a file the host's file tools read, or a chained command, is MINOR, and the finding names the command.

## Read set

You read only:

- your unit directory `<root>/.cad-work/<unit>/`: the brief, the `input/` copies and the files you write;
- every installed file of this plugin and each `resources` path: the eight skill directories `cad-flow`, `cad-record`, `cad-select`,
  `cad-design`, `cad-model`, `cad-verify`, `cad-review` and `cad-release` (every file under each: references, schemas, assets) in the
  installed skills directory that holds your `resources.skills` directories, and the role reference, rubric, schemas, templates,
  `layout` and `policy`. List and search them with the host's file tools, rooted at a skill directory or at that installed skills
  directory; a hit there outside the eight skill directories is no input, never read further. Reading or searching a skill your brief
  does not list is never a boundary defect and never a finding; you LOAD only the `resources.skills` entries
  ([Every role](#every-role) step 2);
- the `resources.tools` binaries, which you execute and never list;
- never needed, and not to be read to be safe: the cad-flow files `stages.md`, `gates.md`, `preflight.md` and `delegation.md` (a repair
  unit reads only the Repair units section of `delegation.md`), and every other `lead-side` link of this file;
- for the [cad-record seal check](../../cad-record/references/validation.md#seal-check) of a `record.semantic` checker only:
  each live `hw/` file that the hash graph reaches from an APPROVED `GATE` or `RELEASE` DR in your `input_inventory.files`,
  with each such DR. Your `check.py` reads these read-only to hash them, and parses a binder among them; the one
  `shasum -a 256 -c` run of that check hashes them again. Never read them with a file tool, copy them, or read them for
  another item.

Never another `.cad-work/` directory (`.cad-work/lead/` with its `preflight.json`, `units.yaml`, `checks.py` and `validator/`, a
`stage-v*` directory, another unit's), any other live `hw/` file, a `boundary.locked` one included (read its input copy), or the
rest of the product tree by listing or search. The tree digest of step 4 hashes the tree without your reading its files,
and uv alone uses the shared cache. Your templates are your only examples of a file's shape, never another unit's files; a path, version
or header the brief does not give is in your skills ([uv commands](#uv-commands)), never a reason to look further.

Reading a file's contents outside the read set is a boundary defect: record each such path in `limits`, and the lead rejects the
delivery ([lead-side: worker delivery Accept](worker-delivery.md#accept) step 8). A names-only listing outside it (a directory listing, a
glob or search that returns names) reads no contents: record it in `limits` as an INFO observation; it changes no grade and rejects nothing.

## Limits

`limits` count from the time step 1 printed: `wall_seconds` of wall time, and `model_turns` of your model replies, a reply that calls
tools counting once. Before each program run and each fetch, and at least every 20 turns, run `date -u +%Y-%m-%dT%H:%M:%SZ` and compare
both with the brief. Once 90% of either is spent, start no new work and deliver what you have; the lead stops a unit still running at
its wall limit ([lead-side: delegation Lost and stalled units](delegation.md#lost-and-stalled-units)).

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
| `unit`, `role`, `input_inventory_sha256` | copied from the [lead-side: worker brief](worker-brief.md)                                                                                                                        |
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

Each role's duty, skills a brief may list, write set, delivery target and role rules are in its role file, which you read with this file:
`agent-roles/<role>.md` beside it, for the `role` of your brief.

| Role                      | Role file                                                                          |
| ------------------------- | ---------------------------------------------------------------------------------- |
| `cad-concept-designer`    | [agent-roles/cad-concept-designer.md](agent-roles/cad-concept-designer.md)         |
| `cad-part-designer`       | [agent-roles/cad-part-designer.md](agent-roles/cad-part-designer.md)               |
| `cad-checker`             | [agent-roles/cad-checker.md](agent-roles/cad-checker.md)                           |
| `cad-mech-dfm-reviewer`   | [agent-roles/cad-mech-dfm-reviewer.md](agent-roles/cad-mech-dfm-reviewer.md)       |
| `cad-sourcing-researcher` | [agent-roles/cad-sourcing-researcher.md](agent-roles/cad-sourcing-researcher.md)   |

The brief lists only the skills that own its methods ([lead-side: worker-brief](worker-brief.md#compose)); a role loads no other. Every
writer also writes the shared uv cache, through uv only ([uv commands](#uv-commands)).

## uv commands

Every uv command of a unit or of the lead uses the one shared cache, runs offline (the sourcing `extract.py` alone excepted) and never
downloads Python. `<cache>` is `<root>/.cad-work/lead/.uv-cache`, warmed by the lead
([lead-side: preflight Warm the uv cache](preflight.md#warm-the-uv-cache)); `<tmp>` is `<root>/.cad-work/<unit>/tmp` for a unit and
`<root>/.cad-work/lead/tmp` for the lead; `<program>` is the absolute path of the PEP 723 script. Every program-running command starts
with the `env` utility, never a bare `TMPDIR=<tmp>` prefix, which some hosts strip before they match a command; a program run's
`> <stdout> 2> <stderr>` redirect follows `<arguments>`:

```sh
env TMPDIR=<tmp> uv lock --cache-dir <cache> --no-python-downloads --offline --script <program>
env TMPDIR=<tmp> uv run --cache-dir <cache> --no-python-downloads --offline --locked --script <program> <arguments>
```

- uv takes the Python 3.12 that preflight found ([lead-side: preflight Probe tools](preflight.md#probe-tools)) from the header's `requires-python`;
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
  lie outside every digest and are never an output or evidence ([lead-side: worker-delivery Accept](worker-delivery.md#accept) step 8),
  except the lite FreeCAD helper `tmp/<check>/freecad-readback.FCScript`, which its result retains as a source file
  ([Every role](#every-role) step 6).
- Schema validation and a YAML load outside a program run only the brief's `resources.tools` argv, prefixed `env TMPDIR=<tmp>`:
  binaries preflight installed, executed read-only, never resolved through `uvx`.

## Rules for every role

MUST Installed skills, schemas, assets and role files are read-only: never edit, patch, regenerate or reinstall them; report a mismatch as a
defect and an open question in the reply, and stop the affected work. MUST Keep missing evidence UNKNOWN with its reason, following the
[cad-record fact contract](../../cad-record/references/record.md#facts); never a default, an invented value or a fallback method. NOT
Delegate, spawn agents, approve, ask the user, or write `hw/`, another unit's scratch or any path outside `boundary.writable`
([lead-side: worker-brief boundary](worker-brief.md#compose)). NOT Repair a locked input or report a UNIT PASS as assembly, stage or
physical evidence.

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
