# Worker delivery

Owner: cad-flow. How the lead accepts or rejects `.cad-work/<unit>/delivery.json` (root `worker-delivery`) and merges accepted bytes.
Workers fill the delivery per [agent-roles Delivery fields](agent-roles.md#delivery-fields); the template is
[worker-delivery YAML](../assets/worker-delivery.yaml).

## Read-only reviewer

The reviewer writes nothing. The lead saves the fenced `json` block of its reply byte-identically as `.cad-work/<unit>/output/review.json`,
then writes `delivery.json` for it: `verdict` from the reply's first line, `build: {status: NOT_APPLICABLE, reason: read-only review}`, one
output `review.json` → the brief's result target, one check entry,
`boundary.method: read-only role; lead tree digest before dispatch and after return` with those two digests, and `ready: true` only for
PASS. The lead also writes the check's bundle manifest with a REVIEW recipe, an inventory of those bytes only. The checks below then apply
unchanged.

## Accept

Run in order; the first failure rejects the delivery with that reason (`units.yaml` state REJECTED; [delegation](delegation.md)).

1. `delivery.json` exists and validates against the worker-delivery root → exit 0.
2. `unit` and `role` equal the brief; `brief_sha256` equals the sha256 of `brief.yaml`; `input_inventory_sha256` equals the brief's digest.
3. Producers (concept designer, part designer, sourcing researcher): `verdict: PASS` and `ready: true`; MINOR and MAJOR are rejected and
   their findings shape the next unit's brief. Evidence roles (checker, reviewer): the verdict grades the subject, so any verdict passes
   this step and a FAIL, UNKNOWN or MAJOR result is published as delivered.
4. `build.status` is SUCCEEDED with a matching log, or NOT_APPLICABLE with a reason. A unit that built geometry also delivers a passing
   `geometry.readback` result ([cad-model Readback](../../cad-model/references/exports.md#readback)); an exit code alone is no success.
5. Every `expected.outputs` target is delivered exactly once and nothing else, except that a `target_pattern` entry takes 0 to `max`
   outputs, each target that pattern with `<sha256>` the output's `sha256` and `<name>` one file name. Each `source` lies under
   `<root>/.cad-work/<unit>/output/`, hashes to its `sha256` and, when the target's [layout](../../cad-record/layout.json) entry names a
   root, validates against it. Programs and locks are never outputs (step 8).
6. Every `expected.checks` entry has a result file that hashes to its sha256, validates against the
   [cad-record check-result root](../../cad-record/schemas/check-result.schema.json), binds this unit's brief and input digests (or the
   brief's `version`), names the delivered `family` and `result`, and has an `implementation` of the right kind: COMMAND whose
   `source_files` name `source/<file>` beside the result target, each with the sha256 of the unit-root file of that name, or REVIEW
   (reviewer only) whose `inspected` entries match the brief inventory.
7. Every log hashes to its sha256.
8. The lead's tree digests before dispatch and after return
   ([cad-record Inventory digest](../../cad-record/references/record.md#inventory-digest)) equal `boundary.before_sha256` and
   `after_sha256`; `violations` is empty; the unit directory holds only `brief.yaml`, `input/**`, `output/**`, `delivery.json` and, at
   its root, `build.py`, `build.py.lock`, `check.py`, `check.py.lock`, `.uv-cache/**` and `__pycache__/**`
   ([agent-roles uv commands](agent-roles.md#uv-commands)); every other unit of the batch still matches its own delivery.
9. Each proposal file exists under `output/proposals/`, hashes to its sha256 and targets a locked or shared `hw/` file; proposals are never
   merged as outputs ([delegation Integrate](delegation.md#integrate)).

Record ACCEPTED with the delivery sha256. A rejected unit is never patched: re-dispatch it as a new unit or stop
([delegation](delegation.md)).

## merge-deliveries

1. Start only after every unit of the batch is ACCEPTED or REJECTED, and copy only ACCEPTED units. No byte of a delivery reaches `hw/`
   before its Accept passes: a rejected unit leaves `hw/` untouched, and a published file is never deleted or overwritten. A result the
   run-checks stop rule discards ([stages run-checks](stages.md#run-checks) step 4) is not merged; its unit stays ACCEPTED.
2. For each accepted output, the result files of a checker or reviewer included: `cp <source> <root>/<target>` with no transform,
   reformat or re-export; then the target's sha256 equals the delivered `sha256`; log `{source, target, sha256}` under the unit.
3. For each accepted COMMAND result, retain its programs ([cad-verify results](../../cad-verify/references/results.md#implementation-kind)):
   `cp <root>/.cad-work/<unit>/<file> <root>/<source_files path>` for every `source_files` entry, `<file>` being that path's file name;
   the copy's sha256 equals the entry; log the triple.
   For a drawing unit ([cad-release draft-drawings](../../cad-release/references/drawings.md#draft-drawings)), also
   `cp <root>/.cad-work/<unit>/build.py` and `build.py.lock` to the `drawing.source` path `<root>/hw/drawings/vN/<SUBJECT>/source/`;
   log each triple with the copy's sha256.
4. A retained source keeps its directory name equal to its sha256 (`hw/manufacturing/sources/<sha256>/<name>`).
5. Defective geometry, annotations or records go back to the producing role as a new unit; a lead edit is a separate, disclosed
   controlled-record edit, never inside merged bytes.
6. UNIT results stay in scratch. The attempt's `delivery.integrity` check re-verifies every merged triple.
