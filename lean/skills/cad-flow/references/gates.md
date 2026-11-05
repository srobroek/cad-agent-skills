# Gates

Owner: cad-flow. `assets/stages.yaml` names each stage's gate and `gate_subject`; this file says how a gate is satisfied and how every
decision is recorded. Decision fields and bindings follow [cad-record's decision schema](../../cad-record/schemas/decision.schema.json); a
published DR is immutable ([cad-record's Acyclic hash graph](../../cad-record/references/record.md#acyclic-hash-graph), step 6).

## Gate kinds and states

| Gate     | Satisfied by                                                                   |
| -------- | ------------------------------------------------------------------------------ |
| `NONE`   | the PASS verdict                                                               |
| `REVIEW` | the PASS verdict, whose required review is PASS                                |
| `HUMAN`  | the PASS verdict plus an APPROVED decision from a USER or AUTHORITY binding it |

No provisional gate exists: a fast check, smoke run or "looks fine so far" opens nothing, and no successor work starts before the gate is
satisfied.

Gate state in every report: `WAITING_FOR_USER` (PASS verdict, no decision yet), `APPROVED`, `REJECTED`, `NOT_REQUIRED` (NONE or REVIEW gate)
or `NOT_PRESENTED` (FAIL or BLOCKED verdict; never put up for approval).

## present-gate

1. Require a PASS verdict for this `vN`; any other verdict goes to the [stop procedure](stages.md#stop-procedure).
2. Present the packet: stage and `vN`; `gate_subject`; the verdict's absolute path, its full 64-hex sha256 and the manifest's full sha256,
   each on its own line; results per family; limits (each UNKNOWN, PLANNED future obligation, waiver, MINOR finding, SAMPLED coverage and
   absent physical evidence); each preview, GLB, drawing and BOM as an absolute path on its own line. A lite build adds
   [Lite packets](#lite-packets).
3. INTENT and VARIANT add the options the verdict assessed
   ([cad-select assess-selection](../../cad-select/references/selection.md#assess-selection) and
   [cad-design compare-variants](../../cad-design/references/concepts.md#compare-variants)): feasible, conditional (with its unknowns) and
   excluded (with the failed hard constraint).
4. End with the exact approval form: `approve <STAGE> <vN>`, plus `choose <option>` for INTENT and VARIANT, or a description of what to
   change. The state is now `WAITING_FOR_USER`.

### Lite packets

Every lite packet opens with the Mode lines of the [SKILL.md OUTPUT](../SKILL.md), adds these items to step 2 and states, just before
the approval form, what that approval covers.

- GROUND, while no geometry exists: the build mode with its source; the proposed form in prose; the part count and arrangement; every
  dimension and measurement criterion; the service class, release scope and exclusions; each geometry-driving UNKNOWN; the checks
  `hw/check-plan.yaml` plans; FreeCAD availability from [preflight.json](preflight.md#freecad) (the executable and version line, or
  absent). Covers: `approve GROUND <vN>` approves exactly this brief, never geometry not yet built.
- RELEASE: the render shown, not only named: the SVG preview displayed in the reply where the host renders images, else opened in the
  system viewer (`open <path>` on macOS, `xdg-open <path>` on Linux) and that stated; the STEP and model-source paths; the measured
  dimensions, solid count and volumes of the accepted geometry results; the FreeCAD version and outcomes `geometry.readback` recorded; the
  independent intent findings of LAYOUT's `intake.use_case`; the exclusions; the proposal's full sha256 beside the verdict and manifest;
  the inventory and REV. Covers: `approve RELEASE <vN>` approves that exact model and package, never fitness for use.

## Record a gate decision

An approval is an explicit statement from the user, or from the authority `product.yaml` names for this scope, that names the stage and `vN`
(and the option for INTENT or VARIANT).

1. Allocate the next unused DR number and save the message exactly as received at `hw/decisions/statements/DR-###.txt`, `test ! -e` first
   (IMMUTABLE in [cad-record's layout](../../cad-record/layout.json)): the file holds the message's own bytes and nothing else, with no
   quotation marks around it and no newline or whitespace added or trimmed. Write it with the host's file-write tool, the message alone as
   the content, or with `printf '%s' '<message>' > <file>`, whose single quotes are shell syntax and inside which each `'` of the message
   is written `'\''`; never `echo` or a heredoc. The reply `approve GROUND v1` is saved as those 17 bytes.
2. Write `hw/decisions/DR-###.yaml` with that number, composed and validated in scratch ([stages Lead scratch](stages.md#lead-scratch)),
   `test ! -e` first: `kind: GATE`, `state: APPROVED` or `REJECTED`,
   `binding: {kind: STAGE, stage, version, verdict: {path, sha256}, verdict_result}` with `verdict_result` copied from that verdict's
   `result` (PASS), `title`, `scope` (the gate subject), `rationale` (the user's reason, or "approved as presented"), `limits` (the
   packet limits), `options` and `choice` for INTENT and VARIANT, `actor`, `time`, and `statement` `{path, transcript_sha256, quote,
   time}`: the saved file, its sha256, a quote from it and its time
   ([cad-record decision schema](../../cad-record/schemas/decision.schema.json)).
3. APPROVED → run the stage's next procedure ID in the same turn: [advance](stages.md#advance) and the successor's `resume-state`, or
   `publish-release` at RELEASE. Never end the turn or ask to continue between recording the decision and that ID. A reply that also says
   `end the session` (`approve INTENT v3 choose <option>, then end the session`) is recorded by steps 1–2 and advances nothing: run
   [preflight Finish](preflight.md#finish) in the same turn, which releases the claim; the next session's `resume-state` starts the
   successor. The approval is the `approve` and `choose` words; `then end the session` is no part of the option.

RELEASE uses `kind: RELEASE` with `binding: {kind: RELEASE_INVENTORY, rev, version, proposal: {path, sha256}, verdict: {path, sha256}}`.
`proposal` hashes the PROPOSAL manifest; `verdict` hashes the PASS RELEASE readiness verdict. The proposal omits `readiness` (forbidden in
PROPOSAL and required in FINAL); `publish-release` runs only after the decision is APPROVED. A REJECTED decision keeps the stage open; the
next attempt follows [change-impact](change-impact.md).

MUST Ask once more with the exact form when a reply lacks the stage and `vN`, names another packet, or is a bare "ok", "continue" or "looks
good"; keep `WAITING_FOR_USER`. MUST Treat a choice the verdict did not assess (an excluded candidate, a new or edited variant) as a
[change](change-impact.md), never as the gate's `choice`. MUST Record a statement scripted by a test harness with `actor.name` starting
`SYNTHETIC` and label it synthetic in every report; it never stands for a product owner. NOT Write an APPROVED or REJECTED decision with
actor LEAD or ROLE, or from silence, a cancelled or timed-out question, a question back, or the lead's own summary. NOT Rewrite a decision:
approve, reject and supersede are new DRs that refer to the old one.

## Decisions outside gates

SELECTION, VARIANT, ORIENTATION, SUPPORT, WAIVER, CHANGE, REQUIREMENT, INTERFACE, ASSUMPTION, QUESTION and IMPORT decisions made during
stage work bind an immutable proposal: write `hw/decisions/proposals/DR-###/<name>` first, then the DR with a PROPOSAL binding
([cad-record decision schema](../../cad-record/schemas/decision.schema.json#/$defs/binding)). A record field that cites a DR approved during
stage work is written before `freeze-inputs`; one approved after an attempt's snapshot is written in the successor attempt's stage work,
never into that snapshot.

The INTENT selection and the VARIANT choice are approved by their gate's `choose` reply
([Record a gate decision](#record-a-gate-decision)); a PROPOSED SELECTION or VARIANT decision that feeds that gate gets no separate approval
request.

APPROVED and REJECTED need one of these explicit statements from the user or the named authority:

| Statement                                                                                                                                         | Decision                                    |
| ------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------- |
| the owner's answer to the question the proposal records, stating the value it proposes (a service class, requirement value, assumption or answer) | APPROVED from that answer; ask nothing more |
| a reply starting `approve DR-###` or `reject DR-###` that names this DR; a waiver adds `waiver WV-###`                                            | APPROVED or REJECTED                        |

1. Save the statement bytes as for a gate ([Record a gate decision](#record-a-gate-decision), step 1), then write the DR.
2. Present every other pending proposal of the stage in one message, each ending with its exact form.

MUST Ask once more with the exact form when a reply names no DR, names another DR or is a bare "ok".

## Waivers

A waiver qualifies readiness for a stated scope; it is never evidence, and the obligation keeps its UNKNOWN or FAIL outcome underneath.

| Obligation                                                                                  | Waiver                                             |
| ------------------------------------------------------------------------------------------- | -------------------------------------------------- |
| family `waiver_policy: FORBIDDEN`, requirement `waiver_policy: FORBIDDEN` or `safety: true` | never                                              |
| safety or regulatory obligation, or a proven unsafe failure                                 | never                                              |
| a missing physical test of a permissible non-safety requirement                             | `UNVALIDATED_TRIAL`: limited, supervised, expiring |
| a bounded non-safety deviation from a non-mandatory criterion                               | `NON_SAFETY_DEVIATION`                             |

1. Allocate the next unused waiver id `WV-###` and write the proposal `hw/decisions/proposals/DR-###/waiver.yaml` with every field of
   [cad-record's common waiver definition](../../cad-record/schemas/common.schema.json#/$defs/waiver), `safety_or_regulatory: false`, and
   its end: `expires`, or `until_change` (the files whose byte change ends it, each with its sha256); an UNVALIDATED_TRIAL always has
   `expires` and `supervision`.
2. Present it; on explicit approval write the WAIVER decision with its `waiver` block whose `id` is that `WV-###` and whose `decision` is
   this DR.
3. PRODUCTION and SAFETY_CRITICAL scope needs the competent AUTHORITY with supplied criteria, allowables and evidence
   ([cad-record Service class and rigor](../../cad-record/references/record.md#service-class-and-rigor)).
4. The verdict maps each covered result to `WAIVED` with its `WV-###` and lists `{id, decision}`
   ([cad-flow write-verdict](stages.md#write-verdict)); every report and the release manifest show it; existence, scope and expiry are
   checked by [cad-record semantic validation](../../cad-record/references/validation.md), and an expired waiver is `EXPIRED`.

A lite build has no waiver: every lite check is `FORBIDDEN` (`lite_stages` in `assets/stages.yaml`), and a material, fit, strength or
manufacturing need is a [promotion to full](change-impact.md#promotion-to-full).

NOT Reclassify the service class or the release scope to avoid an obligation: a concept release stays concept scope.
