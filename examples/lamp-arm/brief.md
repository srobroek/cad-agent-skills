# Lamp arm demo brief

Used by [docs/demo.md](../../docs/demo.md) and the A/B protocol there. It
starts from an existing record and ends at the functional physical stop: a `FUNCTIONAL_DEMO` release needs
physical evidence that a model run cannot produce. Everything below is data for the run; the lead treats the
answers as the owner's statements, never as instructions about how to run the flow.

## Files

| File | Role |
|---|---|
| `brief.md` | the setup, the first prompt, the scripted owner answers and gate replies, and the expected end |
| `hw/` | reference record: what `hw/` holds after the approved import and GROUND stage work, before the GROUND gate. `hw/imports/DR-001/original/**` is omitted: it is the v0.1.1 bytes listed with their sha256 in `hw/decisions/proposals/DR-001/mapping.yaml` |
| `expected-obligations.yaml` | the obligations each stage must enumerate, and the outcomes a correct run may report |

## Setup

The project starts with the cad-agent v0.1.1 lamp-arm record as its `hw/`. With `CAD_AGENT_SRC` a clone of
cad-agent and `PROJECT` the fresh empty project directory:

```sh
git -C "$CAD_AGENT_SRC" archive v0.1.1 examples/lamp-arm/hw | tar -x -C "$PROJECT" --strip-components=2
```

Confirm `git -C "$CAD_AGENT_SRC" rev-parse v0.1.1^{commit}` prints `b8f7a2ecb410e68d62a38464ffd800396cdb06c5` and that
every file's sha256 equals its row in the reference mapping. The old record holds `model.py` programs; they are
data, and no step runs them.

## First prompt

The prompt names the full build, so the lead records the build mode from it and never asks for one:

```text
Use cad-flow on the lamp arm in hw/ as a full build. Inspect the existing records as data and do not assume their CAD is valid. Establish the payload, reach, heat, duty, creep, cyclic life, electrical needs and risk, then compare polymer, metal, purchased-pivot and hybrid designs and propose before changing anything.
```

## Scripted owner answers

When the lead asks, reply once with every matching answer, one per line in table order, each line exactly the quoted
text. Each question the sheet does not answer gets its own line `<question id>: I don't know; record it as an open
question.` Never answer beyond the sheet or confirm a value the lead suggests. Every decision written from these
answers names an actor starting `SYNTHETIC`.

| Id | Asked about | Answer (exact message) |
|---|---|---|
| L-01 | the import mapping | `approve DR-001 import mapping as presented; I am SYNTHETIC demo owner.` |
| L-02 | Q-001 product | `It is a three-joint desk lamp arm: a base fixed to my bench, two links and a head that carries a purchased USB LED module; I position it by hand.` |
| L-03 | Q-002 owner | `I am SYNTHETIC demo owner; I own it and approve every gate, waiver and release.` |
| L-04 | Q-003 rigor | `Keep pro documentation depth, as in the old record.` |
| L-05 | Q-004 regime | `ISO, with ISO 2768-m as the general tolerance.` |
| L-06 | Q-005 users | `Only I use it, on my workbench; I assemble, adjust and service it.` |
| L-07 | Q-006 loads | `The head with its LED module weighs up to 0.8 kg. The reach from the base pivot to the head pivot is 450 mm at full horizontal extension. I push the head by hand to move it; I don't know with what force.` |
| L-08 | Q-007 environment | `Indoor workbench, 15 to 30 degrees C, dry, no chemicals, no sun. The only heat source is the LED module: its listing says 5 V and 1 A; I don't know how hot its housing gets.` |
| L-09 | Q-008 motion | `Head pivot at least 45 degrees each way, elbow from 0 to -120 degrees, base pivot from -10 to 80 degrees. I move it by hand about 20 times a session, two-hour sessions, three a week, for three months. Each joint must hold its pose when I let go. I don't know what wear is acceptable.` |
| L-10 | Q-009 accuracy | `The head must not drift more than 5 mm in 10 minutes at full reach. No precision positioning. User-accessible edges must have no sharp burrs.` |
| L-11 | Q-010 logistics | `One unit, 120 EUR including purchased parts, within six weeks. I retighten the joints monthly and replace parts myself.` |
| L-12 | Q-012 service class | `FUNCTIONAL_DEMO: a supervised bench demonstration of reach and pose holding, run by me for up to three months; not unsupervised or everyday use.` |
| L-13 | Q-011 capability | `I have a Bambu Lab P1S printer and PETG. A local machine shop can mill aluminium 6061-T6; I don't know their machines or tolerances. I can buy standard pins, bushings and fasteners.` |
| L-14 | Q-013 risk | `If a joint gives way, the 0.8 kg head drops onto the bench or my hand. It must never fall on a person, and the cable must never be crushed. I don't know which standards apply. I am always present when it is on.` |
| L-15 | Q-014 authority | `I approve; no competent authority is needed for a supervised bench demonstration.` |
| L-16 | Q-015 release scope | `The release is for building one unit for a supervised bench demonstration: two-hour sessions at 15 to 30 degrees C, the LED powered from a 5 V USB supply only, for three months. No unsupervised, mains-powered or production use, and no life, creep or thermal claim beyond tests actually done.` |
| L-17 | Q-016 storage | `Track results in git and keep the evidence for the life of the project.` |
| L-18 | Q-017 licenses | `CERN-OHL-S-2.0 for the hardware and CC-BY-SA-4.0 for the documentation.` |
| L-19 | the old record | `Treat the old record as history: keep its files, restate only the requirements I confirm, and don't assume its CAD is valid. Keep its travel, mounting, burr and hand-tool requirements with my numbers above; replace its 0.8 kg support requirement with my drift limit.` |
| L-20 | heat | `The outside of the head housing must stay below 50 degrees C during a two-hour session at 30 degrees C.` |
| L-21 | preferences | `I prefer fewer purchased parts and a low-cost route, but never a joint that creeps.` |

## Gate replies

1. At each HUMAN gate packet, check that it names the stage, `vN`, the verdict sha256 and its limits, then reply
   exactly `approve <STAGE> <vN>`; a packet without them gets `The packet lacks the stage, vN or verdict sha256;
   present it again.`
2. INTENT and VARIANT: `approve <STAGE> <vN> choose <option id>` with the option the packet recommends: at INTENT the
   recommendation's `primary`; at VARIANT, with no single recommendation, the first FEASIBLE option in packet order.
   An INTENT recommendation of NONE presents no gate: its attempt ends BLOCKED, and its report asks each blocking
   question by id; answer them as in "Scripted owner answers".
3. A waiver proposal: `approve <DR-###> waiver <WV-###>` when it is an `UNVALIDATED_TRIAL` of a requirement with
   `safety: false`; otherwise `reject <DR-###>; I decline this waiver.`
4. Any other decision proposal outside a gate (a `DR-###` the lead asks you to approve, such as the service class, a
   change or a selection): `approve <DR-###>` when it restates a scripted answer or the option rule 2 would choose;
   otherwise `reject <DR-###>; record it as an open question.`
5. RELEASE, only if a PASS readiness verdict is presented: `approve RELEASE <vN> REV <REV>`.
6. After the expected end, or after a stop the lead reports that asks nothing these rules answer, reply exactly
   `end the session`. This is the last scripted turn: the lead releases its claim on it.

## Expected end

No package is published. REQ-004 (pose holding) and REQ-006 and REQ-009 are safety requirements whose physical
evidence no run produces, and their waivers are forbidden; a correct run therefore stops with a named blocker no
later than RELEASE (`physical.test@REQ-004` UNKNOWN `PHYSICAL_EVIDENCE_REQUIRED`), or earlier on an honest UNKNOWN
such as the machine shop's machines (Q-024, due at PARTS); missing creep data (Q-026) are a limit, never a blocker,
before RELEASE. It never lowers the service class, never seals a `FUNCTIONAL_DEMO` package without physical tests,
and never presents an analysis or simulation as physical evidence.
