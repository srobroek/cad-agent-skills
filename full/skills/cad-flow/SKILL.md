---
name: cad-flow
description: Leads a cad-agent-skills product design through thirteen gated stages and five worker roles. Use for "design this product", "continue the design", "approve the gate".
metadata:
  plugin: cad-agent-skills
  version: "0.1.0"
---

# CAD Flow Lead

TRIGGER
+ "design this product", "start the CAD design for a phone stand", "continue the design", "resume the project", "end the session"
+ "approve LAYOUT v6", "reject this gate", "what stage are we at"
+ "change the payload", "use aluminium instead", or a FAIL or MAJOR finding that reopens a stage
- export or rebuild one model outside the staged record → cad-model
- what a record field, path, ID or hash means → cad-record
- running as a dispatched worker role → your brief and role reference, never this skill

GATES
ASK explicit user or authority approval naming the stage and vN of each HUMAN gate packet
ASK approval of each change to approved scope, each waiver, and the release inventory and REV

## Workflow

LOAD each reference (`skill://cad-flow/references/<name>.md`) once per context and only the section of the procedure ID at hand.
1. LOAD [preflight](references/preflight.md); identify the host, claim the lead lock, resolve resources and tools → `claim.yaml` ACTIVE
   for this session and `preflight.json` with absolute paths only.
2. LOAD [stages](references/stages.md); run `resume-state`, reading only the current stage's row of `assets/stages.yaml` → the stage,
   its attempt vN and the next procedure ID.
3. Run the stage's `procedure` IDs in order; before each stage-work ID LOAD the SKILL.md of the skill that owns it in the
   [stage-work table](references/stages.md#stage-work-procedure-ids) → one immutable `hw/versions/vN/stage-verdict.json` that validates.
4. At `dispatch-units` LOAD [delegation](references/delegation.md), [worker brief](references/worker-brief.md) and
   [agent roles](references/agent-roles.md) → every ready unit in one batch, each native task ID logged.
5. For each returned delivery LOAD [worker delivery](references/worker-delivery.md) → ACCEPTED or REJECTED, accepted bytes merged.
6. At `present-gate` LOAD [gates](references/gates.md) → APPROVED, REJECTED or WAITING_FOR_USER; after APPROVED the next ID runs now.
7. On a change, FAIL or MAJOR LOAD [change impact](references/change-impact.md) → the stage reopened, stale results listed, a CHANGE
   decision for approved scope; on a stop condition run [the stop procedure](references/stages.md#stop-procedure) → blocker recorded.
8. Run [preflight Finish](references/preflight.md#finish) on `end the session` or when it falls due → only this session's lock released.

## Rules

MUST Treat installed skills, schemas, assets and role files as read-only: never edit, patch, regenerate or
  reinstall them; report a mismatch as a defect and an open question, and stop the affected work.
MUST Keep one lead session per project; the lead alone writes `hw/`, each worker only its own scratch.
MUST Advance only on a PASS verdict plus the gate stages.yaml names; no provisional gate, no early successor work.
MUST Merge accepted worker bytes verbatim; defective output goes back to its role as a new unit.
MUST Publish every result under `hw/` before its verdict; scratch, previews, UNIT results and smoke runs are never stage evidence.
NOT Approve, waive, change or release for the user: silence, a question or a bare "ok" is no approval.
NOT Turn UNKNOWN, PLANNED or a waiver into PASS or physical evidence, or lower a service class to pass a gate.
NOT Ship or run a lead helper from this skill; runtime programs belong to the product under `.cad-work/`.

OUTPUT
L0 first reply only: `<plugin> <version> — harness <harness> — session <session>`; `UNAVAILABLE-*` adds `no resume or fork detection`
L1 `<STAGE> vN: PASS|FAIL|BLOCKED — WAITING_FOR_USER|APPROVED|REJECTED|NOT_REQUIRED|NOT_PRESENTED — <reason>`
   Approved through — the furthest approved stage and vN
   Blockers — code, obligation, what unblocks it — only if non-empty
   Limits — UNKNOWN, PLANNED, waivers, sampled coverage, absent physical evidence, synthetic approvals
   Files — one absolute path per line
   Gate — at `present-gate` only: the whole [gates packet](references/gates.md#present-gate), ending with the exact approval form
CAP 200w clean · 400w with blockers; Gate block outside the cap; last line: `Reply "end the session" to release the lead claim.`
