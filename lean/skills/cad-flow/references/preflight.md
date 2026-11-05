# Preflight and the lead claim

Owner: cad-flow. Runs first in every lead session, before any write under `hw/`. Outputs the live claim `.cad-work/lead/active/claim.yaml`
(prose root `lead-claim`, [cad-record prose contract](../../cad-record/references/prose-contract.md#lead-claim)) and
`.cad-work/lead/preflight.json` (scratch, no root). One shared local filesystem is supported; several hosts writing one project are
unsupported.

## Identify the host

Runs before every other preflight command.

1. `harness` is the host this conversation runs in, known from its own system prompt and tool names: `OMP`, `CLAUDE`, `CODEX`,
   `OPENCODE` or `KIRO`. Never infer it from environment variables or the process tree: a shell another host started passes on that
   host's variables.
2. `session` is the native id of this conversation, read only from the row of `harness` by printing that one variable through the host's
   shell tool (`printenv CODEX_THREAD_ID`), never from an eval or REPL kernel, whose environment can be stale, and never by listing or
   searching the environment. Unset, empty or failing the row's check → `UNAVAILABLE-<started stamp>`.

   | `harness`  | `session` source (shell tool)                                                              |
   | ---------- | ------------------------------------------------------------------------------------------ |
   | `OMP`      | `printenv PI_SESSION_ID`; the value appears in `printenv PI_SESSION_FILE`                  |
   | `CLAUDE`   | `printenv CLAUDE_CODE_SESSION_ID`                                                          |
   | `CODEX`    | `printenv CODEX_THREAD_ID`: the thread id `codex resume` takes and its rollout file names |
   | `OPENCODE` | `printenv OPENCODE_SESSION_ID`                                                             |
   | `KIRO`     | none: Kiro exports no session variable; always `UNAVAILABLE-<started stamp>`               |

3. A variable of another row (`PI_*`, `CLAUDE_*`, `CODEX_*`, `OPENCODE_*`) is never `session` and never evidence of `harness`; never
   invent an id. An `UNAVAILABLE-*` session means resume matching and fork detection are unavailable on this host.
4. `isolation` is `NOT_APPLICABLE`, and no `omp` command runs, on every harness except OMP.
5. Harness `OMP` only: `omp config get task.isolation.enabled --json` prints `false` → `isolation: DISABLED`; otherwise stop and ask
   the user to set it, because an isolated worker writes into a clone the lead never reads.
6. Every harness: the first delivery appears at `<root>/.cad-work/<unit>/delivery.json`; a delivery elsewhere stops dispatch.

## Claim the project

1. `root` is `pwd -P` of the product project: the directory that holds, or will hold, `hw/`.
2. `mkdir -p .cad-work/lead/sessions .cad-work/lead/tmp`.
3. Take the lock and write the claim in one shell command, never the exclusive `mkdir` alone:
   `mkdir .cad-work/lead/active && printf '%s\n' <one quoted argument per claim line> > .cad-work/lead/active/claim.yaml`, the claim
   holding `root`, `host`, `harness`, `session`, `owner`, `started` (UTC, quoted), `resumes: []`, `state: ACTIVE`. Check it against the
   `lead-claim` table → every field and rule holds; a claim that fails is corrected in place before any `hw/` write.
4. Failure → read `claim.yaml`:
   - same `session` → append `{session, time}` to `resumes` and continue;
   - another session, or the directory without `claim.yaml` → stop and show the user that claim; write nothing.
5. Recovery only when the operator states the old session has stopped and how they confirmed it. Copy the old claim unchanged to
   `.cad-work/lead/sessions/<old stamp>/claim.yaml`, then write the new claim with
   `recovery: {reference (their confirmation), operator, previous_session, time}`.

NOT Treat the claim's age, a PID or a quiet terminal as proof that its owner stopped.
MUST Before the first `hw/` write of every turn, `.cad-work/lead/active/claim.yaml` exists, has passed step 3's check and names this session
([stages resume-state](stages.md#resume-state) step 1); otherwise stop and write nothing.

## Session directory

`<stamp>` is the claim's `started` without `-` and `:` (`20261103T101500Z`). `.cad-work/lead/sessions/<stamp>/` holds this session's
claim, preflight and unit log at finish. Nothing in `.cad-work/lead/sessions/` is ever deleted.

## Resolve resources

1. Resolve each of the eight skills (cad-flow, cad-record, cad-select, cad-design, cad-model, cad-verify, cad-review, cad-release): use the
   SKILL.md directory loaded by the host (OMP: the path behind `skill://<name>`), resolve it with `realpath`, and test `-f <dir>/SKILL.md`.
2. Test `-f` each derived path: `<cad-record>/schemas/<root>.schema.json` for `common` and the ten schema roots that
   [cad-record's prose contract](../../cad-record/references/prose-contract.md) lists, that prose contract itself,
   [cad-record's layout](../../cad-record/layout.json), `<cad-flow>/assets/stages.yaml`, `<cad-flow>/assets/worker-brief.yaml`,
   `<cad-flow>/assets/worker-delivery.yaml`, the brief templates
   `<cad-record>/assets/contract-fixtures/check-result/valid/{version-pass,unit-prebuild,semantic-aggregate-fail,review-pass}.json`,
   [cad-flow agent-roles](agent-roles.md) and [cad-review's rubric](../../cad-review/references/rubric.md)
   `<cad-review>/references/rubric.md`.
3. Plugin provenance ([cad-record common plugin](../../cad-record/schemas/common.schema.json#/$defs/plugin)): `name` and `version` are the
   `metadata.plugin` and `metadata.version` values in the frontmatter of the resolved `<cad-flow>/SKILL.md`, on every host. Either value
   missing → stop as an installation defect.
4. The host can dispatch all five roles (cad-concept-designer, cad-part-designer, cad-checker, cad-mech-dfm-reviewer,
   cad-sourcing-researcher) by name.
5. No resolved path lies under `<root>/hw/` or `<root>/.cad-work/`.

Run the tests of steps 1, 2 and 5 as one shell command. The stage policy's conformance is checked by `record.semantic` item 17 in every
attempt ([cad-record validation](../../cad-record/references/validation.md#semantic-check)), not here. A failure in steps 1–5 stops the
session as an installation defect: report the path and the observation as an open question for the operator, never repair it
([cad-flow install rule](../SKILL.md#rules)).

## Probe tools

`<cache>` is `<root>/.cad-work/lead/.uv-cache`, the one shared uv cache ([uv commands](agent-roles.md#uv-commands)), `<tmp>` is
`<root>/.cad-work/lead/tmp` and `<tools>` is `<root>/.cad-work/lead/validator`.

| Probe                                                                                                                                                                                    | Required |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| `command -v uv curl` prints one absolute path per tool                                                                                                                                   | yes      |
| `uv --version`                                                                                                                                                                           | yes      |
| `uv python find --cache-dir <cache> --no-python-downloads 3.12`                                                                                                                         | yes      |
| `<tools>/bin/check-jsonschema --version` exits 0                                                                                                                                       | yes      |
| `test "$(<tools>/bin/python -c 'from importlib.metadata import version as v; print(v("check-jsonschema"), v("PyYAML"))')" = '<check-jsonschema pin> <PyYAML pin>'` exits 0              | yes      |
| `curl --version` reports 7.83 or later (`--remove-on-error`; a mid-transfer `--max-filesize` abort needs 8.4)                                                                            | yes      |
| each optional tool a [cad-verify](../../cad-verify/SKILL.md) or [cad-release](../../cad-release/SKILL.md) reference names for a check `hw/check-plan.yaml` already plans; none at GROUND | no       |

Copy both pins from [cad-record Schema validation](../../cad-record/references/validation.md#schema-validation) and
[YAML loader](../../cad-record/references/validation.md#yaml-loader) in this session, never from memory, into the version probe and the
warm-up's `uv pip install` line. An installed version other than its pin fails the probe, and the warm-up below reinstalls the pins.

Run the required probes as one shell command. Every uv command of preflight names `--cache-dir <cache>`, except `uv --version`, which
reads no cache. When a `<tools>` probe fails, the warm-up below installs both tools; then probe again.

Record in `tools[]` `check-jsonschema` with argv `[<tools>/bin/check-jsonschema]` and `yaml-loader` with argv `[<tools>/bin/python]`,
each with the pin the version probe confirmed, `uv` and `curl` with argv `[<the path command -v printed>]`, and `python` with the path the
`uv python find` probe printed, each with its printed version; never a typed or remembered path. Every schema validation and YAML load
of the session, the lead's and each unit's, runs exactly these argv; briefs copy them into `resources.tools`, and a unit only executes
them.

A missing required tool stops with `MISSING_RESOURCE`. A missing optional tool is recorded; its checks become UNKNOWN `UNSUPPORTED_METHOD`,
never a fallback method.

### FreeCAD

Lite build only: when `hw/product.yaml` `build_mode` is KNOWN `LITE`, or right after the owner answers LITE and before GROUND's
`freeze-inputs`. FreeCAD is a project resource of the `geometry.readback` checker unit, run only through its brief's `resources.tools`
argv inside its unit directory ([agent-roles](agent-roles.md)); the lead runs only this probe and never a result.

1. Resolve the console executable: `command -v freecadcmd FreeCADCmd`, else `/Applications/FreeCAD.app/Contents/Resources/bin/freecadcmd`
   when that file exists; take its `realpath`. Never the GUI `FreeCAD` binary, and never install or download FreeCAD without the
   owner's approval.
2. `mkdir -p <tmp>/freecad-home`, then `env -i HOME=<tmp>/freecad-home FREECAD_USER_HOME=<tmp>/freecad-home TMPDIR=<tmp> <freecad> --version`
   from `<tmp>` (the readback's own environment, [cad-model FreeCAD readback](../../cad-model/references/exports.md#freecad-readback)
   step 4), under the shell tool's 60 s timeout → exit 0 and a first line naming FreeCAD and its version (observed:
   `FreeCAD 1.1.4 Revision: 20260928 (Git shallow)`).
3. Success → `tools[]` `{name: freecad, argv: [<realpath>], version: <that first line>}`; this version is a probe observation, never
   evidence. Any failure → `unavailable[]` `{name: freecad, observation}` with what was run and seen. Either way the session continues:
   the GROUND packet shows it ([gates Lite packets](gates.md#lite-packets)), and an absent FreeCAD makes LAYOUT's readback UNKNOWN
   `NOT_RUN` ([stages Lite rows](stages.md#lite-rows)).

### Warm the uv cache

The lead fills `<cache>` with network so that every later uv command, the lead's and each unit's, runs `--offline` against it. The warm
sets are those of [uv commands](agent-roles.md#uv-commands): `yaml` in preflight, and the kernel sets before the session's first unit or
lead program of a stage from CONCEPT on.

Each warm-up is one shell command, the lines below joined with `&&`, run once with network; the host's one escalation for it is
expected. The venv lines run only when a `<tools>` probe failed; the lock and run lines repeat for each warm set due, with
`<tmp>/warm-<set>.py` holding only that set's header:

```sh
env TMPDIR=<tmp> uv venv --cache-dir <cache> --no-python-downloads --clear --python 3.12 <tools>
env TMPDIR=<tmp> uv pip install --cache-dir <cache> --no-python-downloads --compile-bytecode --python <tools>/bin/python check-jsonschema==<pin> pyyaml==<pin>
env TMPDIR=<tmp> uv lock --cache-dir <cache> --no-python-downloads --script <tmp>/warm-<set>.py
env TMPDIR=<tmp> uv run --cache-dir <cache> --no-python-downloads --locked --script <tmp>/warm-<set>.py
```

A warm-up that fails stops with `MISSING_RESOURCE`; a unit is never dispatched to warm its own environment.

## preflight.json

Fields: `root`, `host`, `harness`, `session`, `plugin {name, version}`, `skills [{name, path, uri}]`, `schemas [{name, path}]`, `layout`,
`policy {path, plugin}`, `role_reference`, `rubric`, `templates [{name, path}]` (the asset and fixture templates of Resolve resources step
2), `tools [{name, argv, version}]`, `unavailable [{name, observation}]` (lite build only, [FreeCAD](#freecad)), `isolation`, `time`.
Every path is absolute. Briefs and verdicts copy resource paths and provenance from this file only. The session's first reply follows
the [SKILL.md OUTPUT](../SKILL.md) first-reply line.

## Finish

Run at session end: when the user says `end the session`, alone or in a gate reply ([gates](gates.md#record-a-gate-decision) step 3), or
otherwise states that the session ends. With an `UNAVAILABLE-*` session, also run it before ending every turn, whether it ends
`WAITING_FOR_USER`, on a stop or with a report: a later turn cannot prove it is this session, so it claims anew.

1. Stop this session's units that still run (`.cad-work/lead/units.yaml`); never another session's.
2. Run [stages Stop procedure](stages.md#stop-procedure) step 3 for every attempt that has `hw/versions/vN/manifest.json` and no
   `stage-verdict.json`: each expected obligation without a published result is a `MISSING_RESULT` blocker, and so is the attempt itself
   while `obligations.yaml` is absent. Finish dispatches, runs and publishes nothing else.
3. Copy `claim.yaml` to `.cad-work/lead/sessions/<stamp>/claim.yaml` with `state: RELEASED` and `released` (UTC), and copy `preflight.json`
   and `units.yaml` beside it.
4. Only when the active claim names this session: `rm .cad-work/lead/active/claim.yaml && rmdir .cad-work/lead/active`.
5. Keep every scratch directory, log and session record.
