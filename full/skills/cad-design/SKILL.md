---
name: cad-design
description: Designs product architecture, concept variants, layout, interfaces and parts from the approved selection. Use for "sketch concepts", "lay out the assembly", "design the parts".
---

# CAD Design

TRIGGER
+ "sketch some concepts", "compare the variants", "lay out the assembly", "define the interfaces"
+ "design the parts", "detail the parts", "add the fits and fasteners", "where does the load go"
+ cad-flow runs `specify-interfaces`, `draft-variants`, `compare-variants`, `define-layout`, `design-parts` or `detail-parts`
+ a `cad-concept-designer` or `cad-part-designer` brief lists this skill
- choosing a material, process, supplier or route → cad-select
- writing, building, exporting or reading back geometry → cad-model
- checking clearance, motion, fits, process, load or mass → cad-verify
- drawings, BOM or release package → cad-release; approving a variant or a gate → cad-flow

## Workflow

1. Read the approved scope: the candidate the APPROVED INTENT gate decision chose in
   `hw/manufacturing/selection.yaml`, and the APPROVED SPECIFY requirements and interfaces → per PN its sourcing,
   material profile and route, or the typed UNKNOWN it carries.
2. LOAD references/layout.md (`skill://cad-design/references/layout.md`) for `specify-interfaces` and
   `define-layout`.
3. LOAD references/concepts.md (`skill://cad-design/references/concepts.md`) for `draft-variants`,
   `compare-variants` and the `cad-concept-designer` duty.
4. LOAD references/parts.md (`skill://cad-design/references/parts.md`) for `design-parts`, `detail-parts`, the
   `cad-part-designer` duty and fidelity by stage.
5. Run the procedure under the heading named by the ID, or the role duty your brief names → its outputs at their
   cad-record layout paths, each validating against its cad-record root (exit 0).
6. Hand every model, build and export to cad-model and every criterion or check to cad-verify → this skill writes
   records, briefs and proposals, never a program or a check result.

## Rules

MUST Take each part's sourcing, material profile and route from the approved selection; a PN without one carries a
  typed UNKNOWN with owner and question, never an invented choice.
MUST Name every interface member, datum, critical surface and route target by a semantic feature id that the
  part model witnesses (cad-record references/record.md "IDs").
MUST Size fits, fasteners, threads and inserts from a retained standard or supplier source, never from memory or
  from model geometry.
MUST Route a needed change to a requirement, criterion, interface contract, route or approved variant through
  cad-flow references/change-impact.md; never edit approved scope to make a design pass.
NOT Edit, merge, rank or repair a delivered variant; the comparison cites the unchanged variant bytes.
NOT Model a purchased part as a guessed shape; use its supplier model or a PROXY with sourced bounds.
NOT Treat a fidelity level, a preview or a successful build as a check result.

OUTPUT
L1 `DESIGN <procedure-or-role> <STAGE>: DONE|BLOCKED — <units or records> — <reason>`
   Units — unit id, role and subjects, one per line — only if any
   Unknowns — typed UNKNOWN with owner and question — only if non-empty
   Files — one absolute path per line
CAP 120w clean · 240w with unknowns
