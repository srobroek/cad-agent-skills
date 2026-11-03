# Delegation

Owner: cad-flow. How the lead splits stage work into worker units, dispatches them and integrates their output.
Brief fields: references/worker-brief.md. Acceptance and merge: references/worker-delivery.md. Role duties and
boundaries: references/agent-roles.md.

## Units

- A unit is one role, one stage, one subject set and one brief, in `.cad-work/<unit>/` (`unit` follows the
  cad-record `unit_id` pattern; `lead` and `stage-*` are reserved).
- An unchanged brief resumes the same unit. A changed brief is a new unit id; an id is never reused.
- Never reset, move or delete any unit's scratch.
- `.cad-work/lead/units.yaml` logs each unit: `unit`, `role`, `stage`, `version`, `brief_sha256`, `task_id`
  (native), `state` (`DISPATCHED|DELIVERED|ACCEPTED|REJECTED|STOPPED`), `delivery_sha256`, `reason`, and
  `merged [{source, target, sha256}]`.

## dispatch-units

1. List the stage's independent units from its stage-work procedure. Units of one stage only, and only after its
   predecessor's gate is satisfied.
2. Write and validate each brief (references/worker-brief.md); log its sha256.
3. Take the tree digest (cad-record references/record.md "Inventory digest") and log it for the batch.
4. Dispatch every independent unit in one batch through the host's native subagent call naming the role, with
   a fresh context. The task text is only the role, the absolute brief path and the absolute role reference
   path; never the lead transcript or a forked conversation.
5. Log each returned native task id; wait on and steer only those ids.
6. Write nothing outside `.cad-work/` while a unit of the batch runs: any other write changes the tree digest.

## Integrate

1. After the whole batch returns, take the tree digest again, then accept or reject each delivery
   (references/worker-delivery.md).
2. Merge accepted bytes verbatim (`merge-deliveries`). A rejected delivery goes back to the same role as a new
   unit whose brief states the defect, or the stage stops.
3. A worker proposal is applied only by the lead: inside the open stage's own controlled files the lead edits
   the record itself, discloses the edit and re-dispatches every unit whose inputs changed; touching approved
   scope it is a change (references/change-impact.md).
4. A consequential unknown a delivery reports becomes a record question with owner in the next stage-work step.
5. The stage's VERSION checks run on the integrated version (references/stages.md `run-checks`); a worker PASS
   is never an assembly or stage PASS.

## Boundaries

- Workers never delegate, approve, ask the user, or write `hw/`, installed files or another unit's scratch
  (references/agent-roles.md). A worker question returns to the lead, who answers from the record or asks the
  user, then re-dispatches.
- A static native `.cad-work/**` write allowance is a union over all units, not per-unit isolation; the file-set
  audit in references/worker-delivery.md is the per-unit check.
- Native permission profiles prevent accidental tool calls; they are no operating-system boundary, and an allowed
  program can still write indirectly.
