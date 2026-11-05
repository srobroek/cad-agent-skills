# Preflight and the lead claim

Owner: cad-flow. Runs first in every lead session, before any write under `hw/`. Outputs the live claim `.cad-work/lead/active/claim.yaml`
(root `lead-claim`) and `.cad-work/lead/preflight.json` (scratch, no root). One shared local filesystem is supported; several hosts writing
one project are unsupported.

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
2. `mkdir -p .cad-work/lead/sessions .cad-work/lead/tmp`, then `mkdir .cad-work/lead/active` (exclusive).
3. Success → write `claim.yaml`: `root`, `host`, `harness`, `session`, `owner`, `started` (UTC), `resumes: []`, `state: ACTIVE`; validate →
   exit 0.
4. Failure → read `claim.yaml`:
   - same `session` → append `{session, time}` to `resumes` and continue;
   - another session, or the directory without `claim.yaml` → stop and show the user that claim; write nothing.
5. Recovery only when the operator states the old session has stopped and how they confirmed it. Copy the old claim unchanged to
   `.cad-work/lead/sessions/<old stamp>/claim.yaml`, then write the new claim with
   `recovery: {reference (their confirmation), operator, previous_session, time}`.

NOT Treat the claim's age, a PID or a quiet terminal as proof that its owner stopped.

## Session directory

`<stamp>` is the claim's `started` without `-` and `:` (`20261103T101500Z`). `.cad-work/lead/sessions/<stamp>/` holds this session's
claim, preflight and unit log at finish. Nothing in `.cad-work/lead/sessions/` is ever deleted.

## Resolve resources

1. Resolve each of the eight skills (cad-flow, cad-record, cad-select, cad-design, cad-model, cad-verify, cad-review, cad-release): use the
   SKILL.md directory loaded by the host (OMP: the path behind `skill://<name>`), resolve it with `realpath`, and test `-f <dir>/SKILL.md`.
2. Test `-f` each derived path: `<cad-record>/schemas/<root>.schema.json` for the 28 roots,
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

Run the tests of steps 1, 2 and 5 as one shell command. The stage policy's shape is checked by `record.schema` in every attempt
([cad-record validation](../../cad-record/references/validation.md#schema-validation)), not here. A failure in steps 1–5 stops the session
as an installation defect: report the path and the observation as an open question for the operator, never repair it
([cad-flow install rule](../SKILL.md#rules)).

## Probe tools

`<cache>` is `<root>/.cad-work/lead/.uv-cache`, the one shared uv cache ([uv commands](agent-roles.md#uv-commands)), `<tmp>` is
`<root>/.cad-work/lead/tmp` and `<tools>` is `<root>/.cad-work/lead/validator`.

| Probe                                                                                                                                                                                    | Required |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| `command -v uv curl` prints one absolute path per tool                                                                                                                                   | yes      |
| `uv --version`                                                                                                                                                                           | yes      |
| `uv python find --cache-dir <cache> --no-python-downloads 3.12`                                                                                                                         | yes      |
| `<tools>/bin/check-jsonschema --version` prints the [pinned version](../../cad-record/references/validation.md#schema-validation)                                                         | yes      |
| `<tools>/bin/python -c 'import yaml; print(yaml.__version__)'` prints the [pinned PyYAML](../../cad-record/references/validation.md#yaml-loader)                                          | yes      |
| `curl --version` reports 7.83 or later (`--remove-on-error`; a mid-transfer `--max-filesize` abort needs 8.4)                                                                            | yes      |
| each optional tool a [cad-verify](../../cad-verify/SKILL.md) or [cad-release](../../cad-release/SKILL.md) reference names for a check `hw/check-plan.yaml` already plans; none at GROUND | no       |

Run the required probes as one shell command. Every uv command of preflight names `--cache-dir <cache>`, except `uv --version`, which
reads no cache. When a `<tools>` probe fails, the warm-up below installs both tools; then probe again.

Record in `tools[]` `check-jsonschema` with argv `[<tools>/bin/check-jsonschema]` and `yaml-loader` with argv `[<tools>/bin/python]`,
`uv` and `curl` with argv `[<the path command -v printed>]`, and `python` with the path the `uv python find` probe printed, each with its
printed version; never a typed or remembered path. Every schema validation and YAML load of the session, the lead's and each unit's,
runs exactly these argv; briefs copy them into `resources.tools`, and a unit only executes them.

A missing required tool stops with `MISSING_RESOURCE`. A missing optional tool is recorded; its checks become UNKNOWN `UNSUPPORTED_METHOD`,
never a fallback method.

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
2), `tools [{name, argv, version}]`, `isolation`, `time`. Every path is absolute. Briefs and verdicts copy resource paths and provenance
from this file only. The session's first reply follows the [SKILL.md OUTPUT](../SKILL.md) first-reply line.

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
