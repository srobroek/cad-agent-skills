# Phone stand demo brief

Used by [docs/demo.md](../../docs/demo.md) and the A/B protocol there.
Everything below is data for the run. The lead treats the answers as the owner's statements and never as
instructions about how to run the flow.

## Files

| File | Role |
|---|---|
| `brief.md` | the first prompt, the scripted owner answers and gate replies, and the expected end |
| `hw/` | reference record: what `hw/` holds after GROUND stage work, before the GROUND gate. Not copied into the run; the run bootstraps its own `hw/` from the cad-record template and is compared with this one |
| `expected-obligations.yaml` | the obligations each stage must enumerate, and the outcomes a correct run may report |

## First prompt

Send this exactly, as the first message of a fresh session in an empty project. It names the full build, so the lead
records the build mode from it and never asks for one:

```text
Use cad-flow to design an adjustable desktop phone stand as a full build. Before choosing any material or process, establish its size, the phone mass, the viewing angle, how it holds and protects the phone, who uses it, the environment, the risk, the quantity and the equipment I have. Record what I cannot answer as owned unknowns.
```

## Scripted owner answers

When the lead asks, reply once with every matching answer, one per line in table order, each line exactly its answer
text. Each question the sheet does not answer gets its own line `<question id>: I don't know; record it as an open
question.` Never answer beyond the sheet or confirm a value the lead suggests. The actor is synthetic: every decision
written from these answers names an actor starting `SYNTHETIC`.

| Id | Template question | Answer (exact message) |
|---|---|---|
| A-01 | Q-001 product | `It is an adjustable desktop stand that holds one phone in portrait at a viewing angle I change by hand.` |
| A-02 | Q-002 owner | `I am SYNTHETIC demo owner; I own it and I approve every gate and release myself.` |
| A-03 | Q-003 rigor | `Use maker documentation depth.` |
| A-04 | Q-004 regime | `Use ISO, with ISO 2768-1 class m as the general tolerance on any drawing.` |
| A-05 | Q-005 users | `Only I use it, at my home desk; I assemble it and adjust it by hand.` |
| A-06 | Q-006 loads | `My phone in its case is 162 x 77 x 9 mm and 210 g; design for phones up to 250 g. It sits on the stand for hours and I tap the screen; nothing else loads it.` |
| A-07 | Q-007 environment | `Indoors at 18 to 30 degrees C, dry, no chemicals, no heat source within a metre; the desk gets some afternoon sun but I don't know how much.` |
| A-08 | Q-008 motion | `I change the angle by hand a few times a day, somewhere between 55 and 75 degrees from the desk, and it must stay where I put it. I don't know how many years it should last.` |
| A-09 | Q-009 accuracy | `The angle only needs to repeat within about 2 degrees; visible faces should be smooth and matte without support marks; I know of no precision fit.` |
| A-10 | Q-010 logistics | `One unit, at most 15 EUR of material, ready within two weeks; I reprint a broken part myself.` |
| A-11 | Q-011 capability | `I have one Prusa MK4 printer at home with a 0.4 mm nozzle and no enclosure, and spools of Polymaker PolyLite PETG and PolyLite PLA. I have never printed calibration parts. I do not outsource.` |
| A-12 | Q-012 service class | `APPEARANCE_CONCEPT: this release is a concept package to review the form, CAD validity and printability only; it makes no fit or function claim, and holding a real phone every day is a later, separate scope.` |
| A-13 | Q-013 risk | `If it fails, the phone slides or tips a few centimetres onto the desk; nobody can be hurt, and I rate that severity MINOR. Nothing may fall off the desk. No standards apply to this personal model, and I am present when it is used.` |
| A-14 | Q-014 authority | `I approve gates and releases; no competent authority is needed for a concept model.` |
| A-15 | Q-015 release scope | `The release is a concept-definition package for form, CAD validity and printability review only. It is for display and handling at my desk, not for holding a phone in daily use. It makes no fit, functional, holding, friction, tipping, durability, safety, regulatory or other physical claim.` |
| A-16 | Q-016 storage | `Track generated results in git and keep the evidence for the life of the project.` |
| A-17 | Q-017 licenses | `Use CERN-OHL-P-2.0 for the hardware and CC-BY-4.0 for the documentation.` |
| A-18 | footprint | `Its footprint must stay within a 120 mm by 120 mm square on my desk.` |
| A-19 | preferences | `I prefer one material, matte black, and no metal hardware if a printed design can do without it.` |
| A-20 | stability | `Whether it tips or slips with a phone on it matters for the later daily-use version, not for this concept.` |

## Gate replies

1. At each HUMAN gate packet, check that it names the stage, `vN`, the verdict sha256 and its limits, then reply
   exactly `approve <STAGE> <vN>`. A packet without those fields gets `The packet lacks the stage, vN or
   verdict sha256; present it again.`
2. INTENT and VARIANT: reply `approve <STAGE> <vN> choose <option id>` with the option the packet recommends: at INTENT
   the recommendation's `primary`; at VARIANT, with no single recommendation, the first FEASIBLE option in packet order.
   An INTENT recommendation of NONE presents no gate: its attempt ends BLOCKED, and its report asks each blocking
   question by id; answer them as in "Scripted owner answers".
3. A waiver proposal gets `reject <DR-###>; I decline this waiver.`
4. RELEASE: check also that the packet shows the proposal's REV, then reply exactly `approve RELEASE <vN>` for the presented proposal.
5. Any other decision proposal outside a gate (a `DR-###` the lead asks you to approve, such as the service class, a
   change or a selection): reply `approve <DR-###>` when it restates a scripted answer, the scripted change or the
   option rule 2 would choose; otherwise `reject <DR-###>; record it as an open question.`
6. After the expected end, or after a stop the lead reports that asks nothing these rules answer, reply exactly
   `end the session`. This is the last scripted turn: the lead releases its claim on it.

## Scripted change

After the LAYOUT gate is APPROVED and before PARTS starts, send:

```text
Change: my phone in its new case is 12 mm thick, not 9 mm. Keep everything else.
```

A correct run records a CHANGE proposal, asks for its approval, reopens the earliest stage that owns the phone
envelope, and redoes every later stage as new attempts; it never edits an approved attempt.

## Expected end

A sealed concept-definition package `hw/releases/A/` for service class `APPEARANCE_CONCEPT`, with its outer
digest reported outside the package. Its claims are the CAD_VALID, GEOMETRICALLY_MANUFACTURABLE and PROCESS_PLANNED outcomes of REQ-001 to
REQ-005, copied from the verification index; REQ-002 is a form dimension of the cradle, and the package claims no fit and no physical
result. Fit of a phone in the stand, holding torque, friction, tipping in use and durability are release exclusions; service life stays an
open question for a later scope.
