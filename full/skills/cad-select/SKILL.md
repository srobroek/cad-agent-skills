---
name: cad-select
description:
  Establish a product use case and select traceable material, manufacturing routes, and architecture; use for material, process, sourcing,
  or manufacturing-suitability decisions.
---

# CAD selection

TRIGGER

- The product use case, service class, materials, manufacturing routes, sourcing, or architecture must be assessed.

* CAD geometry without a choice → cad-design; manufacturing verification of a fixed choice → cad-verify.

## Workflow

1. Follow `references/selection.md` → `intake-use-case` in GROUND to establish the use case before comparing candidates. Ask focused
   questions about purpose, users, environment, load magnitude/direction/cycles, motion/duty/life, heat, moisture, chemicals, UV, corrosion,
   accuracy, finish, failure consequence/standards, quantity, budget, lead time, maintenance/repair, equipment, operators and outsourcing.
2. Record each reply whole in one batch at GROUND (`references/selection.md` → `Ask and finish intake` step 2): its facts, volunteered ones
   included, and its requirements, preferences, assumptions and open questions as distinct collections, each criterion one a stage can
   evidence (→ `Write criteria a stage can evidence`). An answer that states a field's value or that it asks nothing settles that field
   (→ `Settle each intake field`); never re-open it or question a release-scope exclusion. After the GROUND gate `hw/requirements.yaml`
   changes only through change impact. Confirm service class separately from documentation rigor; a material-driving or safety-critical
   unknown keeps the candidates it feeds CONDITIONAL.
3. Read `references/selection.md` and `references/materials.md` (OMP: `skill://cad-select/references/selection.md`,
   `skill://cad-select/references/materials.md`). Use the cad-record skill, schema roots and literal absolute paths supplied in the active
   brief; they are read-only.
4. Check every cited material profile, provider and capability against its actual identity, form, process state and service conditions; keep
   unsupported facts UNKNOWN.
5. Follow `references/selection.md` → `select-candidates` in INTENT before sourcing-research units are dispatched to form complete
   material/process/architecture routes per part and contacting pair.
6. After researcher deliveries are merged, follow `references/selection.md` → `assess-selection` in INTENT. Assess a criterion at INTENT
   only when INTENT holds its evidence, else mark it PLANNED with its due stage; a material settles by identity, its consumable form at
   PARTS. Eliminate hard-constraint FAILs before comparing preferences; keep mandatory UNKNOWNs conditional and never let a score revive
   them.
7. Record assessed candidates and the recommendation in the selection record per `references/selection.md` → `Recommend`, plus limits,
   accepted tradeoffs and reversal triggers. A NONE recommendation still ends INTENT in a BLOCKED verdict whose report asks each blocking
   question by its id.
8. Record the assessed selection and immutable proposal before the human gate. Present exact tradeoffs for approval; do not change the
   product design or promote the proposal to a final choice without explicit human approval.

## Rules

MUST Use only names in the record contracts; propose a contract gap to the lead instead of inventing fields, paths, enums or check families.
MUST Treat supplier-typical and standard-minimum profile values as sourced guidance, never allowables or proof of a capability. MUST Choose
transparency requirements only through a purchased transparent part or identified transparent stock with traceable grade, thickness, optical
criterion, supplier and suitable approved process; never recommend FDM for an optical requirement. MUST Keep purchased-component identity
and compatibility checks separate from made-part process checks. NOT Infer achievable tolerance from machine resolution, general-tolerance
labels, a capability template, or a material profile.

OUTPUT Selection record and immutable decision proposal, ready for lead validation; never self-publish or approve them.
