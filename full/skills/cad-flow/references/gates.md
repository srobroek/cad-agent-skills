# Gates

Owner: cad-flow. `assets/stages.yaml` names each stage's gate and `gate_subject`; this file says how a gate is
satisfied and how every decision is recorded. Decision fields and bindings are the cad-record `decision` root;
a published DR is immutable (cad-record references/record.md "Acyclic hash graph" step 6).

## Gate kinds and states

| Gate | Satisfied by |
|---|---|
| `NONE` | the PASS verdict |
| `REVIEW` | the PASS verdict, whose required review is PASS |
| `HUMAN` | the PASS verdict plus an APPROVED decision from a USER or AUTHORITY binding it |

No provisional gate exists: a fast check, smoke run or "looks fine so far" opens nothing, and no successor work
starts before the gate is satisfied.

Gate state in every report: `WAITING_FOR_USER` (PASS verdict, no decision yet), `APPROVED`, `REJECTED`,
`NOT_REQUIRED` (NONE or REVIEW gate) or `NOT_PRESENTED` (FAIL or BLOCKED verdict; never put up for approval).

## present-gate

1. Require a PASS verdict for this `vN`; any other verdict goes to the stop procedure (references/stages.md).
2. Present the packet: stage and `vN`; `gate_subject`; the verdict's absolute path and sha256; the manifest
   sha256; results per family; limits (each UNKNOWN, PLANNED future obligation, waiver, SAMPLED coverage and
   absent physical evidence); each preview, GLB, drawing and BOM as an absolute path on its own line.
3. INTENT and VARIANT add the options the verdict assessed: feasible, conditional (with its unknowns) and
   excluded (with the failed hard constraint).
4. End with the exact approval form: `approve <STAGE> <vN>`, plus `choose <option>` for INTENT and VARIANT,
   or a description of what to change. The state is now `WAITING_FOR_USER`.

## Record a gate decision

An approval is an explicit statement from the user, or from the authority `product.yaml` names for this scope,
that names the stage and `vN` (and the option for INTENT or VARIANT).

1. Save the message bytes unchanged to `.cad-work/lead/sessions/<stamp>/statements/DR-###.txt`
   (references/preflight.md "Session directory").
2. Write `hw/decisions/DR-###.yaml` with the next unused number and `test ! -e` first: `kind: GATE`,
   `state: APPROVED` or `REJECTED`, `binding: {kind: STAGE, stage, version, verdict: {path, sha256},
   verdict_result: PASS}`, `title`, `scope` (the gate subject), `rationale` (the user's reason, or "approved as
   presented"), `limits` (the packet limits), `options` and `choice` for INTENT and VARIANT, `actor`, `time`
   and `statement: {transcript_sha256 (of the saved bytes), quote (the approving sentence), time}`.
3. Validate against the decision root → exit 0.

RELEASE uses `kind: RELEASE` with `binding: {kind: RELEASE_INVENTORY, rev, version, proposal}` naming the
release proposal of the PASS readiness verdict; `publish-release` runs only after it is APPROVED.

A REJECTED decision keeps the stage open; the next attempt follows references/change-impact.md.

MUST Ask once more with the exact form when a reply lacks the stage and `vN`, names another packet, or is a
  bare "ok", "continue" or "looks good"; keep `WAITING_FOR_USER`.
MUST Treat a choice the verdict did not assess (an excluded candidate, a new or edited variant) as a change
  (references/change-impact.md), never as the gate's `choice`.
MUST Record a statement scripted by a test harness with `actor.name` starting `SYNTHETIC` and label it
  synthetic in every report; it never stands for a product owner.
NOT Write an APPROVED or REJECTED decision with actor LEAD or ROLE, or from silence, a cancelled or timed-out
  question, a question back, or the lead's own summary.
NOT Rewrite a decision: approve, reject and supersede are new DRs that refer to the old one.

## Decisions outside gates

SELECTION, VARIANT, ORIENTATION, SUPPORT, WAIVER, CHANGE, REQUIREMENT, INTERFACE, ASSUMPTION, QUESTION and IMPORT
decisions made during stage work bind an immutable proposal: write `hw/decisions/proposals/DR-###/<name>`
first, then the DR with `binding: {kind: PROPOSAL, path, sha256}`. APPROVED and REJECTED need the same explicit
statement as a gate. A controlled-record field that cites an approval is written in the successor attempt's
stage work, never into the approved attempt's snapshot.

## Waivers

A waiver qualifies readiness for a stated scope; it is never evidence, and the obligation keeps its UNKNOWN or
FAIL outcome underneath.

| Obligation | Waiver |
|---|---|
| family `waiver_policy: FORBIDDEN`, requirement `waiver_policy: FORBIDDEN` or `safety: true` | never |
| safety or regulatory obligation, or a proven unsafe failure | never |
| a missing physical test of a permissible non-safety requirement | `UNVALIDATED_TRIAL`: limited, supervised, expiring |
| a bounded non-safety deviation from a non-mandatory criterion | `NON_SAFETY_DEVIATION` |

1. Allocate the next unused waiver id `WV-###` and write the proposal `hw/decisions/proposals/DR-###/waiver.yaml`
   with every field of the cad-record common `waiver`, including `expires`, `supervision` and `until_change` for
   an UNVALIDATED_TRIAL, and `safety_or_regulatory: false`.
2. Present it; on explicit approval write the WAIVER decision with its `waiver` block (cad-record common `waiver`),
   whose `id` is that `WV-###` and whose `decision` is this DR.
3. PRODUCTION and SAFETY_CRITICAL scope needs the competent AUTHORITY with supplied criteria, allowables and
   evidence (cad-record references/record.md "Service class and rigor").
4. The verdict maps each covered result to `WAIVED` with its `WV-###` and lists `{id, decision}`
   (references/stages.md `write-verdict`); every report and the release manifest show it; existence, scope and
   expiry are cad-record semantic checks, and an expired waiver is `EXPIRED`.

NOT Reclassify the service class or the release scope to avoid an obligation: a concept release stays concept scope.
