# Worker brief

Owner: cad-flow. How the lead writes `.cad-work/<unit>/brief.yaml` (root `worker-brief`) before dispatch.
Template: [worker-brief template](../assets/worker-brief.yaml). Role duties, skills, read set and write sets: [agent-roles](agent-roles.md).

## Compose

1. `mkdir .cad-work/<unit>` must succeed; an existing directory is an existing unit ([delegation](delegation.md)). Then
   `mkdir .cad-work/<unit>/tmp`, the unit's temporary directory ([agent-roles uv commands](agent-roles.md#uv-commands)).
2. Copy every `input_inventory.files` entry byte-identically to `.cad-work/<unit>/input/<its project-relative path>`, and write every
   `input_inventory.snapshots` entry at `.cad-work/<unit>/input/<its path>`. A snapshot is a fact the lead owns outside `hw/`, named by a
   logical path, never an absolute path:
   - `lead/attempts.txt`: one line `v<M>` per attempt number found under `hw/versions/`, `hw/checks/` or `.cad-work/stage-v*` when the
     brief is written, the current attempt included, in ascending number order, each line ending in LF. Attempt numbers only grow, so the
     copy an earlier brief held is this file cut after that attempt's own line, and its sha256 is the sha256 of those bytes;
   - `installed/<skill-name>/<path-within-skill>`: a byte copy of the installed file at that path in the skill directory
     `preflight.json` resolved (`installed/cad-record/assets/project-template/hw/product.yaml`).

   Then write `.cad-work/<unit>/input/SHA256SUMS` as the output of one `shasum -a 256` run (Linux: `sha256sum`) over the absolute paths
   of the `files` copies and then the `snapshots` copies, in list order, and paste each `sha256` of the brief from that output, never
   typed ([agent-roles Every role](agent-roles.md#every-role) step 8).
3. Fill the fields below, validate against the worker-brief root → exit 0, record the sha256 `shasum -a 256` prints for it in
   `.cad-work/lead/units.yaml` ([delegation](delegation.md#dispatch-units)). The brief is never edited after dispatch; a change is a new unit.
4. Self-check before dispatch → every item holds; else rewrite the undispatched brief and repeat step 3:
   - Digests: `input_inventory.sha256` recomputes with the set-digest command of
     [cad-record's Inventory digest](../../cad-record/references/record.md#inventory-digest) over the `files` paths;
     `shasum -a 256 -c <root>/.cad-work/<unit>/input/SHA256SUMS` reports every line OK; and its lines carry the brief's `files` and
     `snapshots` sha256 values in list order.
   - Inputs: each input the `input_inventory` row derives for the brief's checks and procedure is in `files` or `snapshots`.
   - Chain, for `record.semantic` and every family whose Method reads stage verdicts, counted against its source: for each line of
     `lead/attempts.txt` before vN that has a `hw/versions/v<M>/`, `files` holds its `manifest.json`, `obligations.yaml` and
     `stage-verdict.json`, whatever its stage (the GROUND attempts at INTENT included); each stage verdict in `files` brings, for every
     result it lists, one `result.json` and the other files
     [cad-record validation Inputs](../../cad-record/references/validation.md#inputs) names for a listed result; and every
     `hw/decisions/DR-*.yaml` comes with the statement and proposal files it hashes.
   - Confirmable hashes: every sha256 an inventoried file records for a lead snapshot (`input/lead/attempts.txt`, `input/installed/…`)
     equals this brief's snapshot of that path or, for an earlier brief's `lead/attempts.txt`, the sha256 of this copy cut after that
     attempt's line (step 2). A hash no copy confirms is a defect to report, never a brief to dispatch.
   - `version` is the current attempt's `vN` for a checker or reviewer and absent for every other role.
   - Outputs: `expected.outputs` names a target or pattern for every subject a producer delivers and, for each check whose result a
     checker or reviewer delivers, its `result.json` target, the target of each artifact its Method names (`artifacts/items.json` for
     `record.semantic`: [cad-record validation Reporting](../../cad-record/references/validation.md#reporting)) and, for a checker, one
     `artifacts/<name>` pattern for the artifacts it names itself.
   - Seeds: a sourcing brief has at least one seed per subject, each on the vendor's own origin
     ([agent-roles Roles](agent-roles.md#roles)), and every seed resolves: one command runs
     `curl -sSL -o /dev/null --max-time 20 -w '%{http_code} %{url_effective}\n' <url>` per seed, and each prints a 2xx code with an
     effective URL on the vendor's origin that still names the subject; a 4xx, 5xx or a redirect elsewhere replaces or drops that seed.
     A subject left without a seed is not briefed: its facts stay typed UNKNOWN with owner and question.
   - `resources.tools` equals the role's entries of the `resources.tools` row, and `limits` holds the row's values.

| Field | Value |
|---|---|
| `unit`, `role`, `stage` | the unit id, one of the five roles (in a lite build `cad-part-designer` or `cad-checker` only: [delegation Lite batches](delegation.md#lite-batches)), the current stage |
| `version` | the published `vN` a checker or reviewer works on; absent for design and sourcing units |
| `task` | one sentence: the subject and the result wanted |
| `subjects` | the PNs, components, interfaces, requirements, cases or operations in scope |
| `seed_urls` | sourcing units only, at least one per subject: every `https` URL on the vendor's own origin that the record, an installed starter profile, the owner or the lead's own search names for the subjects |
| `input_inventory` | `files`: every `hw/` file the unit reads with its sha256, and their set digest `sha256` ([cad-record's Inventory digest](../../cad-record/references/record.md#inventory-digest)); `snapshots`: each lead-owned fact of Compose step 2 the unit reads, with its sha256. Both derive from the brief, never from a guess: each `expected.checks` entry adds every input its method reads ([cad-verify families](../../cad-verify/references/families.md) Method; for `record.semantic`, [cad-record validation Inputs](../../cad-record/references/validation.md#inputs)), and a producer every file its stage-work procedure reads. A unit with a `version` reads the snapshot `hw/versions/vN/**`, never a live file the snapshot holds, the `hw/checks/vN/` bundles its families read and every earlier-version file its method's inputs name, the chain of Compose step 4 included. A checker adds, for each family with an accepted result in an earlier version, the latest such result's `source/check.py` and `source/check.py.lock` ([cad-verify implementation kind](../../cad-verify/references/results.md#implementation-kind) step 2) |
| `limits` | `{wall_seconds, model_turns}` the unit enforces on itself ([agent-roles Limits](agent-roles.md#limits)) and after which the lead stops it ([delegation Lost and stalled units](delegation.md#lost-and-stalled-units)): 2400 and 150 for a checker, reviewer or sourcing unit; 3600 and 200 for a designer unit |
| `resources.plugin` | `{name, version}` copied from [preflight](preflight.md#preflightjson) |
| `resources.skills` | only the skills that own the brief's methods, each with its absolute SKILL.md path and `skill://` URI: cad-record always; the owner of the stage-work procedure ID a producer runs ([stages](stages.md#stage-work-procedure-ids)); cad-model when the unit builds geometry; cad-release when it draws; cad-select for sourcing; cad-verify and the owner of each briefed family's Method ([cad-verify families](../../cad-verify/references/families.md)) when it runs checks; cad-review for the reviewer. The unit loads only these skills and may read any installed file of the plugin ([agent-roles Read set](agent-roles.md#read-set)) |
| `resources.rubric` | `{name: rubric, path, uri}`: the `rubric` path of `preflight.json` and `skill://cad-review/references/rubric.md` ([cad-review's rubric](../../cad-review/references/rubric.md)) |
| `resources.schemas` | every root the unit writes or validates, always `worker-delivery`, and `check-result` when it runs checks |
| `resources.tools` | the role's `preflight.json` `tools[]` entries, copied verbatim ([preflight Probe tools](preflight.md#probe-tools)): `check-jsonschema` and `yaml-loader` for a checker; `check-jsonschema` alone for a designer or sourcing unit, whose role profile denies the loader; none for the reviewer. A lite LAYOUT checker whose `expected.checks` include a `geometry.readback` check also gets the `freecad` entry when preflight has one ([preflight FreeCAD](preflight.md#freecad)); without it the executable is unavailable, which its method records, never a gap the brief fills. No other brief lists `freecad` |
| `resources.templates` | from the `templates` of `preflight.json`: the worker-delivery template for every writer, and the valid check-result fixture of each result kind the unit writes (`version-pass.json`, `unit-prebuild.json`, `semantic-aggregate-fail.json` for `record.semantic`, `review-pass.json` for the reviewer and for a lite LAYOUT checker's intent inspection: [cad-verify families](../../cad-verify/references/families.md#lite-intent-inspection)) |
| `resources.role_reference` | the absolute path and URI of [agent-roles](agent-roles.md) |
| `resources.layout`, `resources.policy` | a unit that runs `record.semantic`: the `layout` and `policy.path` values of `preflight.json` |
| `boundary.project_root` | the claim's `root` |
| `boundary.scratch`, `boundary.writable` | `.cad-work/<unit>`; the role's write set from [agent-roles](agent-roles.md) under `.cad-work/<unit>/`, its `tmp` included, and for every writer the shared uv cache `.cad-work/lead/.uv-cache` ([agent-roles uv commands](agent-roles.md#uv-commands)). A lite geometry checker adds `build.py` and `build.py.lock` when the version holds a model of a kernel other than its `check.py` warm set's ([cad-model exports — geometry.build](../../cad-model/references/exports.md#geometrybuild), Lite) |
| `boundary.locked` | `hw/` files a producer or sourcing unit may not change and proposes changes to: interfaces, assembly, other PNs, criteria; `[]` for a checker or reviewer. A locked file is never read live: its bytes reach the unit as an input copy |
| `expected.outputs` | each `hw/` target with its kind: every subject a producer delivers and each delivered result's `result.json` and artifact targets (Compose step 4 Outputs); never a program, lock, command log (`stdout.txt`, `stderr.txt`) or `source/` path: the lead retains programs, locks and command logs from the unit ([worker-delivery merge-deliveries](worker-delivery.md#merge-deliveries)), while a log a method names under `artifacts/` is an artifact target. A pattern `{target_pattern, kind, max}` takes 0 to `max` outputs ([worker-delivery Accept](worker-delivery.md#accept) step 5): a sourcing brief's retained sources, `hw/manufacturing/sources/<sha256>/<name>` or `hw/parts/<PN>/sources/<sha256>/<name>` with `max` summing to at most 8 per brief; a checker's own-named artifacts, `hw/checks/vN/<check>/<attempt>/artifacts/<name>` with `max` at most 16 per check |
| `expected.checks`, `expected.obligations` | the check-plan checks and obligation IDs the unit must run and cover ([cad-verify check plan](../../cad-verify/references/check-plan.md)), of the families [run-checks](stages.md#run-checks) step 1 gives its role, never a lead family ([stages Lead check program](stages.md#lead-check-program)); a checker or reviewer brief lists both |
| `expected.stop` | the unit's stop conditions beyond the role's own; never one that contradicts a rule of [agent-roles](agent-roles.md) or of a skill the brief lists |
| `expected.report` | `VERDICT line, delivery.json absolute path, findings, unknowns, proposals` |
| `lead.session` | the claim's `session`; the native task id is logged in `units.yaml` only |

Every path in `resources` and `boundary.project_root` is copied from `preflight.json`; never typed, guessed or
resolved from the product directory.
