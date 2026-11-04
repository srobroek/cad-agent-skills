# Preflight and the lead claim

Owner: cad-flow. Runs first in every lead session, before any write under `hw/`. Outputs the live claim `.cad-work/lead/active/claim.yaml`
(prose root `lead-claim`, [cad-record prose contract](../../cad-record/references/prose-contract.md#lead-claim)) and
`.cad-work/lead/preflight.json` (scratch, no root). One shared local filesystem is supported; several hosts writing one project are
unsupported.

## Claim the project

1. `root` is `pwd -P` of the product project: the directory that holds, or will hold, `hw/`.
2. `mkdir -p .cad-work/lead/sessions`, then `mkdir .cad-work/lead/active` (exclusive).
3. Success → write `claim.yaml`: `root`, `host`, `harness`, `session`, `owner`, `started` (UTC), `resumes: []`, `state: ACTIVE`; check it
   against the `lead-claim` table → every field and rule holds.
4. Failure → read `claim.yaml`:
   - same `session` → append `{session, time}` to `resumes` and continue;
   - another session, or the directory without `claim.yaml` → stop and show the user that claim; write nothing.
5. Recovery only when the operator states the old session has stopped and how they confirmed it. Copy the old claim unchanged to
   `.cad-work/lead/sessions/<old stamp>/claim.yaml`, then write the new claim with
   `recovery: {reference (their confirmation), operator, previous_session, time}`.

NOT Treat the claim's age, a PID or a quiet terminal as proof that its owner stopped.

## Session directory

`<stamp>` is the claim's `started` without `-` and `:` (`20261103T101500Z`). `.cad-work/lead/sessions/<stamp>/` holds this session's
approval statements, and at finish its claim, preflight and unit log. Nothing in `.cad-work/lead/sessions/` is ever deleted.

## Resolve resources

1. Resolve each of the eight skills (cad-flow, cad-record, cad-select, cad-design, cad-model, cad-verify, cad-review, cad-release): use the
   SKILL.md directory loaded by the host (OMP: the path behind `skill://<name>`), resolve it with `realpath`, and test `-f <dir>/SKILL.md`.
2. Test `-f` each derived path: `<cad-record>/schemas/<root>.schema.json` for `common` and the ten schema roots that
   [cad-record's prose contract](../../cad-record/references/prose-contract.md) lists, that prose contract itself,
   [cad-record's layout](../../cad-record/layout.json), `<cad-flow>/assets/stages.yaml`, `<cad-flow>/assets/worker-brief.yaml`,
   `<cad-flow>/assets/worker-delivery.yaml` and [cad-flow agent-roles](agent-roles.md).
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

| Probe                                                                                                                                                                                    | Required |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| `uv --version`                                                                                                                                                                           | yes      |
| `uvx --from check-jsonschema==0.39.0 check-jsonschema --version`                                                                                                                         | yes      |
| `uv python find 3.12`                                                                                                                                                                    | yes      |
| each optional tool a [cad-verify](../../cad-verify/SKILL.md) or [cad-release](../../cad-release/SKILL.md) reference names for a check `hw/check-plan.yaml` already plans; none at GROUND | no       |

Run the required probes as one shell command.

A missing required tool stops with `MISSING_RESOURCE`. A missing optional tool is recorded; its checks become UNKNOWN `UNSUPPORTED_METHOD`,
never a fallback method.

## Host isolation

- OMP host only: `omp config get task.isolation.enabled --json` prints `false`; otherwise stop and ask the user to set it, because an
  isolated worker writes into a clone the lead never reads. Every other host runs no `omp` command and records `isolation: NOT_APPLICABLE`.
- Every host: the first delivery appears at `<root>/.cad-work/<unit>/delivery.json`; a delivery elsewhere stops dispatch.

## preflight.json

Fields: `root`, `host`, `harness`, `session`, `plugin {name, version}`, `skills [{name, path, uri}]`, `schemas [{name, path}]`, `layout`,
`policy {path, plugin}`, `role_reference`, `templates`, `tools [{name, argv, version}]`, `isolation`, `time`. Every path is absolute. Briefs
and verdicts copy resource paths and provenance from this file only.

## Finish

1. Stop this session's units that still run (`.cad-work/lead/units.yaml`); never another session's.
2. Copy `claim.yaml` to `.cad-work/lead/sessions/<stamp>/claim.yaml` with `state: RELEASED` and `released` (UTC), and copy `preflight.json`
   and `units.yaml` beside it.
3. Only when the active claim names this session: `rm .cad-work/lead/active/claim.yaml && rmdir .cad-work/lead/active`.
4. Keep every scratch directory, log and session record.
