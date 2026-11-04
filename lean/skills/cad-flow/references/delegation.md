# Delegation

Owner: cad-flow. How the lead splits stage work into worker units, dispatches them and integrates their output. Brief fields:
[worker-brief](worker-brief.md). Acceptance and merge: [worker-delivery](worker-delivery.md). Role duties and boundaries:
[agent-roles](agent-roles.md).

## Units

- A unit is one role, one stage, one subject set and one brief, in `.cad-work/<unit>/` (`unit` follows the
  [cad-record common unit ID](../../cad-record/schemas/common.schema.json#/$defs/unit_id); `lead` and `stage-*` are reserved).
- An unchanged brief resumes the same unit. A changed brief is a new unit id; an id is never reused.
- Never reset, move or delete any unit's scratch.
- `.cad-work/lead/units.yaml` logs each unit: `unit`, `role`, `stage`, `version`, `brief_sha256`, `task_id` (native), `state`
  (`DISPATCHED|DELIVERED|ACCEPTED|REJECTED|STOPPED`), `delivery_sha256`, `reason`, and `merged [{source, target, sha256}]`.

## dispatch-units

1. List every ready unit of the stage: the units its stage-work procedure names
   ([stages Stage-work procedure IDs](stages.md#stage-work-procedure-ids)) and the run-checks units
   ([stages run-checks](stages.md#run-checks) step 1). Units of one stage only, and only after its predecessor's gate is satisfied.
2. Write and validate each brief and pass its self-check ([worker-brief Compose](worker-brief.md#compose)); log its sha256.
3. Take the tree digest ([cad-record's Inventory digest](../../cad-record/references/record.md#inventory-digest)) and log it for the batch.
4. Dispatch every independent unit in one batch through the host's native subagent call naming the role, with a fresh context. The task text
   is only the role, the absolute brief path and the absolute role reference path; never the lead transcript or a forked conversation.
   Where the call has a history switch, set it to none: Codex `spawn_agent` with `agent_type: <role>`, a new `task_name` per unit and
   `fork_turns: "none"` (the default `all` forks the lead's whole history).
5. Log each returned native task id; wait on and steer only those ids. Wait for every unit of the batch within the same turn, with the
   host's foreground or blocking wait where it offers one: OpenCode and Claude Code subagent calls run in the foreground, never with
   `background: true` or `run_in_background: true`. Never end a turn or present a gate while a dispatched unit still runs. A unit lost to
   a turn end is `STOPPED` and goes out again as a new unit ([Integrate](#integrate) step 2).

   | `harness`  | native `task_id`                                                         |
   | ---------- | ------------------------------------------------------------------------ |
   | `OMP`      | the task id the `task` call returns                                      |
   | `CLAUDE`   | the `agentId` the Agent (Task) call returns                              |
   | `CODEX`    | the `spawn_agent` agent id, else its `/root/<task_name>` path            |
   | `OPENCODE` | the subagent's child session id                                          |
   | `KIRO`     | the subagent's sub-session id (`subSessionId`), else its tool call id    |

   An id the host does not show the lead is logged `UNAVAILABLE: <what the call returned>`.
6. Write nothing outside `.cad-work/` while a unit of the batch runs: any other write changes the tree digest. Meanwhile the lead runs its
   own families into `.cad-work/stage-vN/checks/` ([stages run-checks](stages.md#run-checks) step 1).

## Integrate

1. After the whole batch returns, take the tree digest again, then accept or reject each delivery ([worker-delivery](worker-delivery.md)).
2. Merge accepted bytes verbatim ([merge-deliveries](worker-delivery.md#merge-deliveries)). A rejected delivery goes back to the same role
   as a new unit, or the stage stops: a new unit id and `.cad-work/<unit>/`, a new brief that states the defect, and a new native subagent
   call with a fresh context per step 4. Never send the rework to a returned task (a follow-up, message, resume or steer of its id; on
   Codex `followup_task` or `send_message`).
3. A worker proposal is applied only by the lead: inside the open stage's own controlled files the lead edits the record itself, discloses
   the edit and re-dispatches every unit whose inputs changed; touching approved scope is a [change](change-impact.md).
4. A consequential unknown a delivery reports becomes a record question with owner in the next stage-work step.
5. The stage's VERSION checks run on the integrated version ([stages run-checks](stages.md#run-checks)); a worker PASS is never an assembly
   or stage PASS.

## Boundaries

- Workers never delegate, approve, ask the user, or write `hw/`, installed files or another unit's scratch ([agent-roles](agent-roles.md)).
  A worker question returns to the lead, who answers from the record or asks the user, then re-dispatches.
- A static native `.cad-work/**` write allowance is a union over all units, not per-unit isolation; the file-set audit in
  [worker-delivery](worker-delivery.md) is the per-unit check.
- Native permission profiles prevent accidental tool calls; they are no operating-system boundary, and an allowed program can still write
  indirectly.
- A host that denies a command this contract requires, to a worker or to the lead, is a host or plugin defect. Record the exact denied
  command, the host's denial text and the installed role or profile path, stop the affected unit (`STOPPED`), and report it as a
  `MISSING_RESOURCE` blocker with an open question for the operator. Never ask the owner to edit an installed role or profile file
  ([install rule](../SKILL.md#rules)).
