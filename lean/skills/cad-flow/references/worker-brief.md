# Worker brief

Owner: cad-flow. How the lead writes `.cad-work/<unit>/brief.yaml` (prose root `worker-brief`,
[cad-record prose contract](../../cad-record/references/prose-contract.md#worker-brief)) before dispatch.
Template: [worker-brief template](../assets/worker-brief.yaml). Role duties, skills and write sets: [agent-roles](agent-roles.md).

## Compose

1. `mkdir .cad-work/<unit>` must succeed; an existing directory is an existing unit ([delegation](delegation.md)).
2. Copy every file the unit reads byte-identically to `.cad-work/<unit>/input/<its project-relative path>`.
3. Fill the fields below, check them against the `worker-brief` table → every field and rule holds, record its sha256 in
   `.cad-work/lead/units.yaml` ([delegation](delegation.md#dispatch-units)). The brief is never edited after dispatch; a change is a new unit.

| Field | Value |
|---|---|
| `unit`, `role`, `stage` | the unit id, one of the five roles, the current stage |
| `version` | the published `vN` a checker or reviewer works on; absent for design and sourcing units |
| `task` | one sentence: the subject and the result wanted |
| `subjects` | the PNs, components, interfaces, requirements, cases or operations in scope |
| `input_inventory` | every inventoried `hw/` file with its sha256, and their set digest ([cad-record's Inventory digest](../../cad-record/references/record.md#inventory-digest)) |
| `resources.plugin` | `{name, version}` copied from [preflight](preflight.md#preflightjson) |
| `resources.skills` | the role's skills from [agent-roles](agent-roles.md), each with its absolute SKILL.md path and `skill://` URI |
| `resources.rubric` | `{name: rubric, path, uri}`: the `rubric` path of `preflight.json` and `skill://cad-review/references/rubric.md` ([cad-review's rubric](../../cad-review/references/rubric.md)) |
| `resources.schemas` | every schema root the unit writes or validates, and `check-result` when it runs checks; never a prose root (its table is in the cad-record skill the role loads) |
| `resources.role_reference` | the absolute path and URI of [agent-roles](agent-roles.md) |
| `resources.layout`, `resources.policy` | a unit that runs `record.semantic`: the `layout` and `policy.path` values of `preflight.json` |
| `boundary.project_root` | the claim's `root` |
| `boundary.scratch`, `boundary.writable` | `.cad-work/<unit>`; the role's write set from [agent-roles](agent-roles.md) |
| `boundary.locked` | `hw/` files the unit may not change: interfaces, assembly, other PNs, criteria |
| `boundary.read_only` | other paths it may read |
| `expected.outputs` | each `hw/` target with its kind, including each check's `result.json` target; never a program, lock or `source/` path: the lead retains programs and locks from the unit root ([worker-delivery merge-deliveries](worker-delivery.md#merge-deliveries)) |
| `expected.checks`, `expected.obligations` | the check-plan checks and obligation IDs the unit must run and cover ([cad-verify check plan](../../cad-verify/references/check-plan.md)) |
| `expected.stop` | the unit's stop conditions beyond the role's own |
| `expected.report` | `VERDICT line, delivery.json absolute path, findings, unknowns, proposals` |
| `lead.session` | the claim's `session`; the native task id is logged in `units.yaml` only |

Every path in `resources` and `boundary.project_root` is copied from `preflight.json`; never typed, guessed or
resolved from the product directory.
