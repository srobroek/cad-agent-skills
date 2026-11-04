---
name: cad-flow
description: Leads a cad-agent-skills product design through thirteen gated stages and five worker roles. Use for "design this product", "continue the design", "approve the gate".
metadata:
  plugin: cad-agent-skills
  version: "0.1.0"
---

# CAD Flow Lead

TRIGGER
+ "design this product", "start the CAD design for a phone stand", "continue the design", "resume the project"
+ "approve LAYOUT v6", "reject this gate", "what stage are we at"
+ "change the payload", "use aluminium instead", or a FAIL or MAJOR finding that reopens a stage
- export or rebuild one model outside the staged record → cad-model
- what a record field, path, ID or hash means → cad-record
- running as a dispatched worker role → your brief and role reference, never this skill

GATES
ASK explicit user or authority approval naming the stage and vN of each HUMAN gate packet
ASK approval of each change to approved scope, each waiver, and the release inventory and REV

## Workflow

1. LOAD [preflight](references/preflight.md) (`skill://cad-flow/references/preflight.md`); claim the lead lock, resolve
   resources → `claim.yaml` ACTIVE for this session and `preflight.json` with absolute paths only.
2. LOAD [stages](references/stages.md) (`skill://cad-flow/references/stages.md`); run `resume-state` against
   `assets/stages.yaml` → the stage, its attempt vN and the next procedure ID.
3. Run the stage's `procedure` IDs in order, loading the stage `owner` skill for stage-work IDs
   → one immutable `hw/versions/vN/stage-verdict.json` that validates.
4. At `dispatch-units` LOAD [delegation](references/delegation.md) (`skill://cad-flow/references/delegation.md`),
   [worker brief](references/worker-brief.md) (`skill://cad-flow/references/worker-brief.md`) and [agent roles](references/agent-roles.md)
   (`skill://cad-flow/references/agent-roles.md`) → one batch, each native task ID logged.
5. For each returned delivery LOAD [worker delivery](references/worker-delivery.md) (`skill://cad-flow/references/worker-delivery.md`)
   → ACCEPTED or REJECTED, accepted bytes merged verbatim.
6. At `present-gate` LOAD [gates](references/gates.md) (`skill://cad-flow/references/gates.md`) → an APPROVED or
   REJECTED decision, or the stage stays WAITING_FOR_USER.
7. On a change, FAIL or MAJOR LOAD [change impact](references/change-impact.md) (`skill://cad-flow/references/change-impact.md`)
   → the stage reopened, stale results listed, a CHANGE decision when approved scope changes.
8. On a stop condition run [the stop procedure](references/stages.md#stop-procedure) → blocker recorded, user told, no further writes.
9. Finish per [preflight](references/preflight.md) → claim copied into the session directory, only this session's lock released.

## Rules

MUST Treat installed skills, schemas, assets and role files as read-only: never edit, patch, regenerate or
  reinstall them; report a mismatch as a defect and an open question, and stop the affected work.
MUST Keep one lead session per project; the lead alone writes `hw/`, each worker only its own scratch.
MUST Advance only on a PASS verdict plus the gate stages.yaml names; no provisional gate, no early successor work.
MUST Merge accepted worker bytes verbatim; defective output goes back to its role as a new unit.
MUST Publish every result under `hw/` before its verdict; scratch, previews, UNIT results and smoke runs are
  never stage evidence.
NOT Approve, waive, change or release for the user: silence, a question or a bare "ok" is no approval.
NOT Turn UNKNOWN, PLANNED or a waiver into PASS or physical evidence, or lower a service class to pass a gate.
NOT Ship or run a lead helper from this skill; runtime programs belong to the product under `.cad-work/`.

OUTPUT
L1 `<STAGE> vN: PASS|FAIL|BLOCKED — WAITING_FOR_USER|APPROVED|REJECTED|NOT_REQUIRED|NOT_PRESENTED — <reason>`
   Approved through — the furthest approved stage and vN
   Blockers — code, obligation, what unblocks it — only if non-empty
   Limits — UNKNOWN, PLANNED, waivers, sampled coverage, absent physical evidence, synthetic approvals
   Files — one absolute path per line
CAP 200w clean · 400w with blockers
