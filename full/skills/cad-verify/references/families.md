# Check families

Owner: cad-verify. One row per registered family (cad-record `common.schema.json#/$defs/check_family`, 54 families):
the anchor that names its obligations, whether `hw/check-plan.yaml` carries it, who produces the result, and the
method owner. Result shape, outcome codes and facets: references/results.md. Scope items: references/check-plan.md
"Expected scope".

- **Plan** `plan`: the check plan carries the check, criterion and cases. `fixed`: no check-plan entry; the check id
  is the family with `.` and `_` replaced by `-` (`geometry.identity` → `geometry-identity`), except
  `drawing.coverage` → `drawing-<subject in lowercase>` and `release.integrity` → `package-integrity`.
- **Producer**: `lead` (cad-flow references/stages.md `run-checks`), `checker` (`cad-checker` unit), `reviewer`
  (`cad-mech-dfm-reviewer` unit). A method owned outside cad-verify is run exactly as its owner states; this table
  adds only the anchor and the outcome rule.
- Every row: a method that cannot run, an input that is missing or a criterion that is UNKNOWN gives UNKNOWN with
  the matching code, never PASS.
- **Facet**: a `plan` family takes the check-plan facet. A `fixed` family records `CAD_VALID`, except
  `geometry.manufacture` (`GEOMETRICALLY_MANUFACTURABLE`); its outcome counts toward a requirement facet in the
  verification index only when it names that requirement.

## Record, intake and selection

| Family | Anchor | Plan | Producer | Method | FAIL when |
|---|---|---|---|---|---|
| `record.schema` | `@product` | fixed | lead | cad-record references/validation.md "Schema validation", one command per layout-mapped file | a command exits non-zero |
| `record.semantic` | `@product` | fixed | checker | cad-record references/validation.md "Semantic check", items 1–14 as separate outcomes | an item is violated |
| `record.ownership` | `@product` | fixed | lead | every `hw/` file matches a cad-record `layout.json` entry; every write since the predecessor came from the lead or a merged delivery | an unclassified file or a write outside ownership |
| `record.inputs` | `@product` | fixed | lead | recompute the input inventory digest (cad-record references/record.md "Inventory digest"); for each CARRIED_FORWARD result compare every input byte (cad-flow references/change-impact.md "Carry-forward") | a digest or byte differs |
| `delivery.integrity` | `@product` | fixed | lead | recompute every merged `{source, target, sha256}` triple (cad-flow references/worker-delivery.md `merge-deliveries`) | a triple differs |
| `intake.use_case` | `@product` | fixed | checker | every `product.yaml` intake domain is KNOWN or a typed UNKNOWN with owner and question; service class KNOWN with its DR (cad-select references/selection.md `intake-use-case`) | a domain is absent or a material- or safety-driving fact is defaulted; UNKNOWN `REQUIRED_UNKNOWN` when it is UNKNOWN |
| `requirements.completeness` | `@product` | fixed | checker | references/check-plan.md `specify-requirements` | per that procedure |
| `interfaces.completeness` | `@product` | fixed | checker | cad-design references/layout.md `specify-interfaces` (SPECIFY, FREEZE) and `define-layout` (LAYOUT): every interface has members, functions, datums, fit, clearance, play, variation, contact, environment and access as KNOWN or typed UNKNOWN | a field is absent or a critical one is defaulted |
| `check_plan.completeness` | `@product` | fixed | checker | references/check-plan.md `plan-checks` | per that procedure |
| `selection.feasibility` | `@product` | fixed | checker | cad-select references/selection.md `assess-selection`: each candidate's class follows from its hard outcomes; only FEASIBLE candidates are ranked; the recommendation is FEASIBLE or labelled CONDITIONAL | a score or preference revives an EXCLUDED or CONDITIONAL candidate, or a hard FAIL is recommended |
| `selection.sourcing` | `@<PN>` per part citing a supplier, stock or purchase; `@product` at INTENT | fixed | checker | each cited identity, grade, thickness and rating matches a retained source quote; each retained source's directory name equals its sha256 | a cited fact contradicts its source; UNKNOWN when unsourced or undated |
| `material.properties` | `@<PN>.<component>` per made component; `@product` at INTENT | fixed | checker | cad-select references/materials.md: every property a criterion or case uses matches the component's grade, form, process state, orientation, temperature and conditioning; strength uses an ALLOWABLE basis | density used as strength, typical used as allowable, another grade or state transferred |
| `purchase.identity` | `@<PN>` per BUY part; `@<PN>.<component>/<operation>` per PURCHASED operation | fixed | checker | references/process-checks.md "Purchased" | per that section |
| `variant.integrity` | `@product` | fixed | checker | CONCEPT: each variant's `variant.yaml` and geometry equal the merged delivery bytes, state feasibility per hard constraint and list unknowns. VARIANT: every variant is byte-identical to its CONCEPT version and the comparison cites only those | a byte differs, a hard constraint is unassessed or a variant was edited |

## Geometry and assembly

| Family | Anchor | Plan | Producer | Method | FAIL when |
|---|---|---|---|---|---|
| `geometry.build` | `@assembly`; `@<PN>` per part with geometry | fixed | checker | cad-model references/exports.md `geometry.build` | per that heading |
| `geometry.readback` | `@assembly`; `@<PN>` per part with geometry | fixed | checker | cad-model references/exports.md `geometry.readback` | per that heading |
| `geometry.export` | `@<PN>` per exported part; `@assembly` | fixed | checker | cad-model references/exports.md `geometry.export` | per that heading |
| `geometry.identity` | `@<PN>.<component>` | fixed | checker | cad-model references/exports.md `geometry.identity` | per that heading |
| `geometry.manufacture` | `@<PN>` per MAKE or MADE_TO_DRAWING part | fixed | checker | cad-model references/exports.md `geometry.manufacture` | per that heading |
| `geometry.interference` | `@assembly`; `@IF-###` per intended interference fit | plan | checker | references/geometry-motion.md `geometry.interference` | per that heading |
| `geometry.clearance` | `@assembly`; `@IF-###[#<feature>]` per clearance criterion | plan | checker | references/geometry-motion.md `geometry.clearance` | per that heading |
| `motion.sweep` | `@case.<case>` per MOTION case | plan | checker | references/geometry-motion.md `motion.sweep` | per that heading |
| `motion.play` | `@case.<case>` per MOTION case with `play_extremes`; `@IF-###` stating play | plan | checker | references/geometry-motion.md `motion.play` | per that heading |
| `motion.access` | `@op.<operation>` per operation or service task needing access | plan | checker | references/geometry-motion.md `motion.access` | per that heading |
| `assembly.operations` | `@op.<operation>` | plan | checker | references/geometry-motion.md `assembly.operations` | per that heading |
| `preview.render` | `@assembly`; `@<PN>`; `@product` at CONCEPT | fixed | checker | cad-model references/exports.md `preview.render` | per that heading |

## Process and route

| Family | Anchor | Plan | Producer | Method | FAIL when |
|---|---|---|---|---|---|
| `capability.route` | `@<PN>.<component>/<operation>` per made operation (every route kind except PURCHASED) | plan | checker | references/process-checks.md `capability.route` | per that heading |
| `optical.route` | `@<PN>.<component>` per optical need; `@product` at INTENT | plan | checker | references/process-checks.md `optical.route` | per that heading |
| `process.fdm` … `process.other` (13) | `@<PN>.<component>/<operation>[#<feature>]` per operation of the mapped route kind | plan | checker | references/process-checks.md, the heading named by the family | per that heading |

The 13 process families: `process.fdm`, `process.resin`, `process.cnc`, `process.laser`, `process.waterjet`,
`process.plasma`, `process.sheet_forming`, `process.joining`, `process.casting`, `process.moulding`,
`process.extrusion`, `process.secondary`, `process.other`; the route-kind mapping is references/process-checks.md
"Route kind to family".

## Fits, mass and loads

| Family | Anchor | Plan | Producer | Method | FAIL when |
|---|---|---|---|---|---|
| `fit.limits` | `@IF-###[#<feature>]` per stated fit or tolerance class | plan | checker | references/fits.md `fit.limits` | per that heading |
| `fit.stack` | `@case.<case>` per STACK case | plan | checker | references/fits.md `fit.stack` | per that heading |
| `mass.properties` | `@case.<case>` per MASS case | plan | checker | references/mass-loads.md `mass.properties` | per that heading |
| `load.hand` | `@case.<case>` per LOAD case with `method: HAND` | plan | checker | references/mass-loads.md `load.hand` | per that heading |
| `load.fea` | `@case.<case>` per FEA case | plan | checker | references/fea.md `load.fea` | per that heading |

## Documents, review, physical and release

| Family | Anchor | Plan | Producer | Method | FAIL when |
|---|---|---|---|---|---|
| `drawing.coverage` | `@<PN>` or `@assembly` per REQUIRED drawing | fixed | checker | cad-release references/drawings.md `drawing.coverage` | per that heading |
| `drawing.preview` | `@<PN>` or `@assembly` per REQUIRED drawing | fixed | checker | cad-release references/drawings.md `drawing.preview` | per that heading |
| `bom.derivation` | `@product` | fixed | checker | cad-release references/bom.md `bom.derivation` | per that heading |
| `verification.index` | `@product` | fixed | checker | re-derive every row by references/check-plan.md `index-verification` from the cited results and compare with `verification.yaml` and the CSV | a row, outcome or cited hash differs, or a row invents an outcome |
| `review.mechanical_dfm` | `@assembly` | plan | reviewer | cad-review references/review.md `review.mechanical_dfm`, graded by cad-review references/rubric.md | the review verdict is MINOR or MAJOR |
| `physical.test` | `@REQ-###` per requirement with a PHYSICALLY_VALIDATED facet | plan | checker | references/results.md "Physical evidence" | the test result misses its criterion; UNKNOWN `PHYSICAL_EVIDENCE_REQUIRED` while no qualifying test exists |
| `release.inventory` | `@product` | fixed | checker | cad-release references/release.md `release.inventory` | per that heading |
| `release.integrity` | `@product` | fixed | lead, inside cad-release references/release.md `publish-release` | cad-release references/release.md `release.integrity` | per that heading |
