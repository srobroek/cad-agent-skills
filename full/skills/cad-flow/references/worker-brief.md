# Worker brief

Owner: cad-flow. How the lead writes `.cad-work/<unit>/brief.yaml` (root `worker-brief`) before dispatch.
Template: [worker-brief template](../assets/worker-brief.yaml). Role duties, skills and write sets: [agent-roles](agent-roles.md).

## Compose

1. `mkdir .cad-work/<unit>` must succeed; an existing directory is an existing unit ([delegation](delegation.md)).
2. Copy every `input_inventory.files` entry byte-identically to `.cad-work/<unit>/input/<its project-relative path>`, then write
   `.cad-work/<unit>/input/SHA256SUMS`: one `<sha256>  <absolute path of the copy>` line per entry, in `files` order, two spaces apart.
3. Fill the fields below, validate against the worker-brief root → exit 0, record its sha256 in
   `.cad-work/lead/units.yaml` ([delegation](delegation.md#dispatch-units)). The brief is never edited after dispatch; a change is a new unit.
4. Self-check before dispatch → every item holds; else rewrite the undispatched brief and repeat step 3:
   - `input_inventory.sha256` recomputes with the set-digest command of
     [cad-record's Inventory digest](../../cad-record/references/record.md#inventory-digest) over the `files` paths, and
     `shasum -a 256 -c <root>/.cad-work/<unit>/input/SHA256SUMS` reports every line OK;
   - `version` is the current attempt's `vN` for a checker or reviewer and absent for every other role;
   - `expected.outputs` names a target or pattern for every subject a producer delivers and each expected check's `result.json`;
   - a sourcing brief has `seed_urls`, empty only when the host gives the role a search tool;
   - `resources.tools` equals the `tools[]` entries of `preflight.json` the unit runs.

| Field | Value |
|---|---|
| `unit`, `role`, `stage` | the unit id, one of the five roles, the current stage |
| `version` | the published `vN` a checker or reviewer works on; absent for design and sourcing units |
| `task` | one sentence: the subject and the result wanted |
| `subjects` | the PNs, components, interfaces, requirements, cases or operations in scope |
| `seed_urls` | sourcing units only: every `https` URL the record, an installed starter profile or the owner names for the subjects |
| `input_inventory` | every inventoried `hw/` file the unit reads with its sha256, and their set digest ([cad-record's Inventory digest](../../cad-record/references/record.md#inventory-digest)). A unit with a `version` gets exactly `hw/versions/vN/**` and the `hw/checks/vN/` bundles its families read, never a live file the snapshot holds or an earlier version's bundle, with two additions: a `record.semantic` unit adds every earlier `hw/versions/v*/{manifest.json,obligations.yaml,stage-verdict.json}`, every `hw/decisions/DR-*.yaml` and the statement and proposal files they hash; a checker adds, for each family with an accepted result in an earlier version, the latest such result's `source/check.py` and `source/check.py.lock` ([cad-verify implementation kind](../../cad-verify/references/results.md#implementation-kind) step 2) |
| `resources.plugin` | `{name, version}` copied from [preflight](preflight.md#preflightjson) |
| `resources.skills` | only the skills that own the brief's methods, each with its absolute SKILL.md path and `skill://` URI: cad-record always; the owner of the stage-work procedure ID a producer runs ([stages](stages.md#stage-work-procedure-ids)); cad-model when the unit builds geometry; cad-release when it draws; cad-select for sourcing; cad-verify and the owner of each briefed family's Method ([cad-verify families](../../cad-verify/references/families.md)) when it runs checks; cad-review for the reviewer |
| `resources.rubric` | `{name: rubric, path, uri}`: the `rubric` path of `preflight.json` and `skill://cad-review/references/rubric.md` ([cad-review's rubric](../../cad-review/references/rubric.md)) |
| `resources.schemas` | every root the unit writes or validates, always `worker-delivery`, and `check-result` when it runs checks |
| `resources.tools` | the `preflight.json` `tools[]` entries the unit runs, copied verbatim: `check-jsonschema` for every role except the reviewer, `yaml-loader` for a checker ([preflight Probe tools](preflight.md#probe-tools)) |
| `resources.templates` | from the `templates` of `preflight.json`: the worker-delivery template for every writer, and the valid check-result fixture of each result kind the unit writes (`version-pass.json`, `unit-prebuild.json`, `semantic-aggregate-fail.json` for `record.semantic`, `review-pass.json` for the reviewer) |
| `resources.role_reference` | the absolute path and URI of [agent-roles](agent-roles.md) |
| `resources.layout`, `resources.policy` | a unit that runs `record.semantic`: the `layout` and `policy.path` values of `preflight.json` |
| `boundary.project_root` | the claim's `root` |
| `boundary.scratch`, `boundary.writable` | `.cad-work/<unit>`; the role's write set from [agent-roles](agent-roles.md) |
| `boundary.locked` | `hw/` files the unit may not change: interfaces, assembly, other PNs, criteria |
| `boundary.read_only` | other paths it may read |
| `expected.outputs` | each `hw/` target with its kind, including each check's `result.json` target; never a program, lock or `source/` path: the lead retains programs and locks from the unit root ([worker-delivery merge-deliveries](worker-delivery.md#merge-deliveries)). A sourcing brief names retained sources by pattern instead: `{target_pattern, kind, max}` with `hw/manufacturing/sources/<sha256>/<name>` or `hw/parts/<PN>/sources/<sha256>/<name>`, `max` summing to at most 8 per brief ([worker-delivery Accept](worker-delivery.md#accept) step 5) |
| `expected.checks`, `expected.obligations` | the check-plan checks and obligation IDs the unit must run and cover ([cad-verify check plan](../../cad-verify/references/check-plan.md)) |
| `expected.stop` | the unit's stop conditions beyond the role's own |
| `expected.report` | `VERDICT line, delivery.json absolute path, findings, unknowns, proposals` |
| `lead.session` | the claim's `session`; the native task id is logged in `units.yaml` only |

Every path in `resources` and `boundary.project_root` is copied from `preflight.json`; never typed, guessed or
resolved from the product directory.
