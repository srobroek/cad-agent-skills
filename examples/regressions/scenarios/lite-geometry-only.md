# A LITE build makes no material or manufacturing claim

Scenario `lite-geometry-only` · stages `LAYOUT`, `RELEASE` · rules: cad-record references/record.md "Build mode";
cad-flow SKILL.md disclosure line.

## Situation

The owner chose a lite build of the one-piece phone stand and mentioned in passing that it will be printed in PETG
later. The part designer writes the PETG profile into the LITE part, and the release proposal adds a
`GEOMETRICALLY_MANUFACTURABLE` claim for the retaining lip because the geometry checks passed.

## Files

| File | Schema | Role |
|---|---|---|
| [`invalid/part/lite-geometry-only-material.yaml`](../invalid/part/lite-geometry-only-material.yaml) | exit 1 (`expected-errors.yaml`) | the LITE part naming a material profile |
| [`valid/part/lite-geometry-only-body.yaml`](../valid/part/lite-geometry-only-body.yaml) | exit 0 | the LITE part: model, component and requirement-linked functions only |
| [`invalid/release-manifest/lite-geometry-only-printable-claim.json`](../invalid/release-manifest/lite-geometry-only-printable-claim.json) | exit 1 (`expected-errors.yaml`) | the LITE proposal claiming a manufacturability facet |
| [`valid/release-manifest/lite-geometry-only-proposal.json`](../valid/release-manifest/lite-geometry-only-proposal.json) | exit 0 | the LITE proposal: CAD_VALID PASS claims and the disclosure line |

## Procedure under test

- cad-design part design of a LITE part and cad-record references/record.md "Build mode"
- cad-release lite release (`propose-release`) and its claims
- cad-record references/validation.md item 18 (build-mode discriminators)

## Expected

1. Both invalid files exit 1; the lead corrects them in scratch and never publishes them.
2. The PETG remark stays the owner's statement in the record; no LITE part, check or claim names a material, route or
   manufacturability facet.
3. A material or manufacturing claim needs promotion to FULL (a CHANGE that reopens GROUND), never a waiver.

## Regression fails when

- a LITE part carries a material profile, sourcing, route or drawing field
- a LITE package claims a facet other than CAD_VALID, or omits the disclosure line
