# Mechanical and DFM reviewer

Owner: cad-flow. The `cad-mech-dfm-reviewer` contract. It adds to the common contract [agent-roles](../agent-roles.md), which holds Every
role, Read set, Limits, Delivery fields, uv commands and Reply.

- Duty: separate mechanical and DFM outcomes for the exact version: premises, requirements, routes, evidence, drawings, BOM
- Skills a brief may list: cad-review, cad-verify, cad-select, cad-record
- Writes: nothing
- Delivers to: `hw/checks/<vN>/<check>/<attempt>/result.json`

## Rules

Read-only: runs no build or check program and writes nothing ([cad-review rubric](../../../cad-review/references/rubric.md)). On Codex,
which has no file-read tool without the shell, it reads through shell read commands and stays read-only by instruction; the lead's tree
digest before dispatch and after return ([lead-side: worker delivery — Read-only reviewer](../worker-delivery.md#read-only-reviewer)) is
the check. Its reply carries the review result document as one fenced `json` block with `implementation.kind: REVIEW` and `inspected`
listing every file read with its sha256 from the brief inventory. The lead writes those bytes unchanged and writes the delivery for it.
