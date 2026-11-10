# Release

Owner: cad-release. This file owns revision promotion, the release ceiling, the release proposal, package
completeness, the staged copy, the package integrity check and publication. Gate and waiver decisions are cad-flow
references/gates.md; paths are cad-record `layout.json` (`version.release_proposal`, `release.*`); manifest
fields are the `release-manifest` root; the hash order is cad-record references/record.md "Acyclic hash graph"
step 8; anchors and producers of `release.inventory` and `release.integrity` are
[cad-verify Check families](../../cad-verify/references/families.md). `vF` below is the approved FREEZE version
and `vM` the RELEASE attempt. A LITE build releases the PASS LAYOUT version `vL` instead: [Lite release](#lite-release).

## Release ceiling

A package is published only after a PASS RELEASE verdict. Which physical evidence each service class needs, and
where an `UNVALIDATED_TRIAL` waiver may stand in for a missing physical test, is
[cad-verify Evidence by service class](../../cad-verify/references/check-plan.md#evidence-by-service-class) and
[cad-verify Physical evidence](../../cad-verify/references/results.md#physical-evidence); this table states what the
published package is.

| Release scope service class | The package is |
|---|---|
| `APPEARANCE_CONCEPT` | a concept-definition package: form, CAD validity and planned process only; `exclusions` name every functional, physical, safety and regulatory claim |
| `FIT_ASSEMBLY`, `FUNCTIONAL_DEMO`, `WORKING_PROTOTYPE` | a qualified package when every due `physical.test` is PASS on physical evidence; with an approved non-safety `UNVALIDATED_TRIAL` waiver, a trial package whose `scope.trial` states the waiver DR, envelope, supervision and expiry |
| `PRODUCTION`, `SAFETY_CRITICAL` | a qualified package only with the criteria, allowables, evidence and approval of the competent AUTHORITY `product.yaml` names; never by waiver |
| `UNKNOWN` | none |

- What a safety requirement needs at each service class, and the APPEARANCE_CONCEPT exclusion of every safety claim, is the
  [cad-verify Evidence by service class](../../cad-verify/references/check-plan.md#evidence-by-service-class) row. A due safety
  obligation without that evidence leaves RELEASE BLOCKED and publishes nothing.
- The service class and release scope are the ones GROUND recorded. Another scope is a CHANGE from GROUND
  (cad-flow references/change-impact.md), never an edit at RELEASE.
- A demonstration run ends in a concept-definition package, a lite prototype package, an approved non-safety trial
  package, or a stop whose blocker is named; never in a functional or physical pass.
- A LITE build is `APPEARANCE_CONCEPT`, and its package is a lite prototype package ([Lite release](#lite-release)). It claims
  CAD validity and the requested form and dimensions only, as its checks observed them, plus STEP import in the recorded
  FreeCAD version. It names no planned process. Its guide and `scope.exclusions` carry the disclosure line of
  cad-flow SKILL.md OUTPUT. A material, fit, strength, process or manufacturing claim needs promotion to a full build
  (cad-flow references/change-impact.md), never a lite waiver.

## promote-revisions

FREEZE, before `freeze-inputs`; nothing is promoted after FREEZE.

1. Baseline: the record copy in the newest sealed FULL package, `hw/releases/<REV>/record/` of a FINAL manifest
   without `build_mode: LITE` (legacy packages included), or none. A lite package is never a revision baseline:
   its parts stay on draft `X` revisions.
2. For every PN in `assembly.yaml`: a part whose `part.yaml`, model and retained sources differ from the
   baseline copy, or that has no baseline, takes the next released letter of `[A-HJ-NPRTUVWY]` after its last
   released letter (`A` when it has none). An unchanged part keeps its released letter.
3. Write the new `rev` into `hw/parts/<PN>/part.yaml` and every occurrence of that PN in `assembly.yaml`.
4. Output: one line per PN `old → new` or `unchanged`, carried into the FREEZE gate packet. A part left on an
   `X` revision blocks FREEZE (`record.semantic`).

## propose-release

RELEASE, after `index-verification` has published the `vM` verification index and its `verification.index` result;
geometry is the FREEZE version's. A LITE build runs [Lite propose-release](#lite-propose-release) instead.

1. Confirm the chain: the predecessor FREEZE verdict `vF` is PASS with an APPROVED GATE decision. Confirm
   `product.yaml` `service.class`, `release_scope` and `license` are KNOWN; an UNKNOWN one is a due UNKNOWN for
   `release.inventory`: stop with `DUE_UNKNOWN` and its question.
2. REV: the next letter of `[A-HJ-NPRTUVWY]` unused by every `hw/releases/*`, every `rev` in
   `hw/versions/*/release-proposal.json` and every RELEASE decision; a rejected or failed REV stays consumed.
3. Build guide: fill `<skill-dir>/assets/build-guide.md` from the frozen record and the `vM` verification index into
   `hw/docs/release-<REV>/build-guide.md`; every statement comes from the record, an UNKNOWN prints UNKNOWN, a
   physical test prints its `vM` index outcome. License texts the owner approved sit beside it.
4. Planned payload, one entry `{path, source, sha256, bytes, role}` per file, sorted by `path`:

   | Package path | Source | Role |
   |---|---|---|
   | `definition/**` | `hw/versions/vF/definition/**` (GLB is visualization only, as the guide says) | DEFINITION |
   | `manufacturing/<PN>/**` | `hw/versions/vF/manufacturing/<PN>/**`, except `manufacturing.yaml` | MANUFACTURE |
   | `drawings/<SUBJECT>/**` | `hw/drawings/vF/<SUBJECT>/**` | DRAWING |
   | `record/version/**` | `hw/versions/vF/{manifest.json,stage-verdict.json,record/**,source/**}` | RECORD |
   | `record/release/manifest.json` | `hw/versions/vM/manifest.json`, which the RELEASE results bind | RECORD |
   | `record/decisions/**` | every `hw/decisions/DR-###.yaml` and cited proposal the chain GROUND to `vF` names | RECORD |
   | `checks/<vX>/<check>/<attempt>/**` | every result bundle a verdict of that chain lists, and every `vM` result bundle published before this step | CHECK |
   | `bom.yaml`, `bom.csv` | `hw/bom/vF/` | BOM |
   | `verification.yaml`, `verification.csv` | `hw/verification/vM/` | VERIFICATION |
   | `docs/build-guide.md` | `hw/docs/release-<REV>/build-guide.md` | DOCUMENT |
   | `docs/LICENSE-*` | the approved license texts | LICENSE |

   A retained source whose redistribution right is not granted is never copied: it goes to `exclusions` with
   its citation, sha256 and the restriction. Scratch, previews and other notes are never payload.
5. `claims`: from `hw/verification/vM/verification.yaml`
   ([cad-verify `index-verification`](../../cad-verify/references/check-plan.md#index-verification)), which holds the
   frozen outcomes and this attempt's `physical.test` outcomes: one claim per requirement and outcome, its facets
   grouped, the outcome copied unchanged (PASS, WAIVED or UNKNOWN; nothing is future at RELEASE). A FAIL row, or a
   `verification.index` result other than PASS, blocks the proposal.
6. `scope`: `service_class`, `purpose`, `operating_limits` and `exclusions` copied from `product.yaml`; `trial`
   only when an `UNVALIDATED_TRIAL` waiver is in use, copied from its WAIVER decision.
7. `frozen` (`vF`, its manifest and verdict hashes, its GATE decision), `decisions` (every DR the package
   cites, with hashes), `waivers`, `licenses` (owner-approved values; each third-party item with its source and
   `redistributed`), `exclusions` (every cited item not packaged, with its reason).
8. Write `hw/versions/vM/release-proposal.json` with `phase: PROPOSAL`, `test ! -e` first → it validates
   against the `release-manifest` root; written once.
9. Dispatch the `release.inventory` check ([below](#releaseinventory)) on the proposal; its result is published
   before `accept-results`.

## release.inventory

The method of family `release.inventory`, check id `release-inventory`, obligation `release.inventory@product`,
`facet: CAD_VALID`, run by a `cad-checker` unit at RELEASE on the proposal and every source it names. One outcome
per criterion:

1. Every planned source exists under `hw/`, outside scratch, and matches its sha256 and bytes.
2. The payload holds what the release ceiling row needs: definition STEP of the assembly and every made part,
   the manufacturing outputs each route names, sheets and coverage for every REQUIRED drawing subject, the BOM
   bundle of `vF` and the verification bundle of `vM`, the record and decisions, every cited check bundle, the
   build guide and the license texts.
3. `claims` equal the `vM` verification index; no FAIL row exists; no UNKNOWN, PLANNED or WAIVED row is upgraded.
4. `scope` equals `product.yaml` and the GROUND service class; `trial` is present exactly when an
   `UNVALIDATED_TRIAL` waiver is used, and that waiver is approved, in scope and unexpired.
5. Licenses are owner-approved; a third-party item is copied only with its redistribution right.
6. `exclusions` list every cited item that is not packaged, with its reason.
7. REV is unused; package paths are unique, relative and safe, and none is `release-manifest.json` or under
   `acceptance/`.

A LITE build ([Lite release](#lite-release)) replaces criteria 2 and 4 and adds 8 to 10:

- Criterion 2: the payload is exactly the [Lite payload](#lite-payload) rows. It has no `manufacturing/`, `drawings/`,
  `bom.*` or STL path, and no `vM` result, readiness verdict, RELEASE decision or integrity result.
- Criterion 4: `build_mode` is `LITE` and equals the `vM` snapshot product's `build_mode`. `frozen` names `vL`, a PASS
  LAYOUT verdict, with its manifest and verdict sha256 and no `decision`. `scope` equals `product.yaml`, with service
  class `APPEARANCE_CONCEPT` and no `trial`, and `waivers` is empty.
- Criterion 8: this check re-derives the claims itself from the cited accepted `vL` results and reads no
  `verification.index` result. Every CAD_VALID row of the approved requirements is PASS or NOT_APPLICABLE, and no
  criterion or subject is missing. `claims` hold exactly the requirements with a PASS row.
- Criterion 9: every packaged STEP has a cited `vL` `geometry.readback` result PASS. Its retained
  `artifacts/freecad-readback.json`
  ([cad-model exports — FreeCAD readback](../../cad-model/references/exports.md#freecad-readback)) is
  `complete: true` and lists that STEP's sha256. `preview.render` is PASS at `@assembly` and every `@<PN>`, and its
  result names each packaged SVG's sha256. A missing render or FreeCAD output, or a readback UNKNOWN `NOT_RUN`, is
  FAIL.
- Criterion 10: the guide holds the lite notice line byte-equal to the one in `<skill-dir>/assets/build-guide.md`,
  and only the sections that template keeps for a LITE build. Every value equals its cited record or result, and each
  PN prints its draft `X` revision. The guide states nothing about making, purchasing, BOM, service, material, fit,
  strength or manufacturing.

The result is PASS only when every outcome is PASS; a missing fact is UNKNOWN `MISSING_INPUT`.

## publish-release

Only after the RELEASE decision is APPROVED and binds `RELEASE_INVENTORY {rev, version: vM, proposal, verdict}`
naming this proposal file and the PASS `vM` readiness verdict, each with its sha256. The post-copy check is not
a second verdict: it never changes the RELEASE verdict or its readiness.
A LITE build runs [Lite publish-release](#lite-publish-release).

1. Recheck: `test ! -e hw/releases/<REV>`; recompute every planned source sha256. A mismatch stops with
   `HASH_MISMATCH`; a frozen input that must change is a CHANGE and a new version.
2. Stage: `mkdir .cad-work/stage-vM/release-<REV>` (it must not exist; else step 7). Copy each planned source
   byte for byte to its package path, following no symlink, then copy the approved RELEASE decision to
   `record/decisions/`, the `vM` readiness verdict to `record/release/stage-verdict.json` and the
   `release.inventory` result bundle to `checks/vM/<check>/<attempt>/`. Rehash every staged file against its
   expected sha256.
3. Integrity: run `release.integrity` (below) on the staged copy and write its result to
   `acceptance/package-integrity.json` in the staging directory. A result other than PASS publishes nothing.
4. Final manifest: the proposal's fields with `phase: FINAL`, plus `readiness` (the `vM` stage verdict path and
   sha256), `approval {decision, file}`, `inventory` (every staged file, including the step 2 copies and
   `acceptance/package-integrity.json` with role ACCEPTANCE, excluding `release-manifest.json`) and
   `integrity {path, sha256, result: PASS}`. It validates against the `release-manifest` root and every
   inventory sha256 equals the staged bytes; otherwise publish nothing.
5. Seal: write `release-manifest.json` last into the staging directory, `mkdir -p hw/releases`, confirm
   `test ! -e hw/releases/<REV>`, then rename the staging directory to `hw/releases/<REV>` with one `mv` on the
   same filesystem. The REV is sealed once `hw/releases/<REV>/release-manifest.json` exists.
6. Report: the package path, the outer digest (`shasum -a 256` of `hw/releases/<REV>/release-manifest.json`,
   reported in the session report only), integrity PASS, the ceiling row, claims by outcome, waivers, scope
   and exclusions. Prune nothing.
7. Interrupted: a staging directory without a sealed REV resumes only when every staged file equals its
   expected sha256 and no other file exists; otherwise stop with `HASH_MISMATCH`, keep the staging directory
   and report. A sealed REV is never written again; another package is a new attempt with a new REV.

## release.integrity

The method of family `release.integrity`, check `package-integrity`, attempt `a1`, obligation
`release.integrity@product`, `facet: CAD_VALID`. The lead runs it, like `delivery.integrity`, as a hash audit of
the staged copy against the approved proposal and the RELEASE decision: `producer_role: lead`, `kind: COMMAND`
with every `shasum -a 256` (or `sha256sum`) command and its exit code. The result binds `VERSION` (`vM` and its
manifest digest) and carries `release {rev, approval, planned_inventory}` with the proposal's path and sha256.

1. Every planned entry exists at its package path with its sha256 and bytes.
2. The only other files are the step 2 copies of [publish-release](#publish-release): the approved decision, the
   readiness verdict whose sha256 that decision binds, and the `release.inventory` bundle, whose manifest that
   verdict hashes and whose files that manifest lists, each with that sha256 (this result is written after the audit).
   In a LITE build they are the bound extras of [Lite publish-release](#lite-publish-release), each with the sha256
   that binds it there.
3. No symlink, no unsafe path, no file outside the staging directory.
4. `artifacts` list every staged file with its sha256, excluding `acceptance/package-integrity.json` and
   `release-manifest.json`.

The result is PASS only when every outcome is PASS.

## Lite release

A LITE build releases `vL`, the PASS LAYOUT version, from the RELEASE attempt `vM`. The frozen `hw/product.yaml`
`build_mode` is KNOWN `LITE`. No FREEZE, BOM or drawing stage precedes it. Its parts stay on draft `X` revisions, and
the package holds geometry, renders, records and checks only. The procedure order is the lite RELEASE row of cad-flow
`assets/stages.yaml`, and the hash graph is cad-record references/record.md "Acyclic hash graph" step 8. Unlike a
full release, the guide and PROPOSAL come before any RELEASE check:

1. `vM` freezes the record without new geometry; the inventoried source is `vL`, never a FREEZE version.
2. cad-verify `index-verification` publishes the `vM` verification bundle from the accepted `vL` results. Then
   [Lite propose-release](#lite-propose-release) writes the guide and the PROPOSAL. Both read only bytes published
   before them, and both plan no RELEASE result.
3. `run-checks` sends every RELEASE checker unit out in one batch over those published bytes, the
   `verification.index` and `release.inventory` units included; no unit reads a pending result. Every required
   result PASSes before readiness.
4. The readiness verdict binds every accepted RELEASE result, the `vM` obligations and manifest, and the `vL`
   verdict. The RELEASE decision binds the proposal and that verdict (`RELEASE_INVENTORY`, cad-flow
   references/gates.md).
5. [Lite publish-release](#lite-publish-release) stages the payload and the extras the readiness verdict and
   approval bind, and `release.integrity` checks each extra against the hash that binds it. The FINAL manifest
   inventories every staged file except itself and is written last.

### Lite payload

| Package path | Source | Role |
|---|---|---|
| `definition/assembly.step`, `definition/parts/<PN>.step` | `hw/versions/vL/definition/` (STEP AP214) | DEFINITION |
| `definition/assembly.glb` | `hw/versions/vL/definition/assembly.glb` (visualization only, as the guide says) | DEFINITION |
| `docs/previews/**` | `hw/versions/vL/previews/**` (renders, never drawings) | DOCUMENT |
| `record/version/**` | `hw/versions/vL/{manifest.json,stage-verdict.json,obligations.yaml,record/**,source/**}` | RECORD |
| `record/chain/<vX>/**` | `hw/versions/<vX>/{manifest.json,stage-verdict.json,obligations.yaml}` of each earlier version in the chain from `vL` back to GROUND | RECORD |
| `record/release/manifest.json` | `hw/versions/vM/manifest.json` | RECORD |
| `record/decisions/**` | every DR of that chain with its statement and cited proposals, at their paths under `hw/decisions/` | RECORD |
| `checks/<vX>/<check>/<attempt>/**` | every result bundle a verdict of that chain lists, the `vL` FreeCAD readback output included | CHECK |
| `verification.yaml`, `verification.csv` | `hw/verification/vM/` | VERIFICATION |
| `record/verification/bundle-manifest.json` | `hw/verification/vM/bundle-manifest.json` | VERIFICATION |
| `docs/build-guide.md` | `hw/docs/release-<REV>/build-guide.md` | DOCUMENT |
| `docs/LICENSE-*` | the approved license texts | LICENSE |

### Lite propose-release

RELEASE of a LITE build, after `index-verification` has published the `vM` verification bundle and before `run-checks`.

1. Chain: the predecessor LAYOUT verdict `vL` is PASS. Its gate is NONE, so it has no gate decision. Its chain back to
   GROUND is unbroken and ends in an APPROVED GROUND gate decision. Its accepted results hold `geometry.readback` PASS
   with complete FreeCAD output, and `preview.render` PASS, at `@assembly` and every `@<PN>`. Otherwise propose
   nothing: the missing obligation is the blocker. `product.yaml` service class must be `APPEARANCE_CONCEPT`, and
   `release_scope` and `license` must be KNOWN; an UNKNOWN one stops with `DUE_UNKNOWN` and its question.
2. REV: as [propose-release](#propose-release) step 2; lite and full packages consume one letter sequence.
3. Build guide: fill the LITE build sections of `<skill-dir>/assets/build-guide.md` into
   `hw/docs/release-<REV>/build-guide.md`. Take the values from the `vM` record snapshot, the accepted `vL` results and
   the `vM` verification index; the guide cites no RELEASE result. Put the approved license texts beside it.
4. Planned payload: one entry `{path, source, sha256, bytes, role}` per [Lite payload](#lite-payload) file, sorted by
   `path`. A source without a redistribution right goes to `exclusions`, as in [propose-release](#propose-release)
   step 4.
5. `claims`: from `hw/verification/vM/verification.yaml`, one per requirement with a PASS row, its facets grouped,
   outcome PASS. A FAIL, UNKNOWN, PLANNED or WAIVED row blocks the proposal; a NOT_APPLICABLE row makes no claim.
6. `build_mode: LITE`. `frozen` holds `vL` with its manifest and verdict hashes, and no `decision`. `scope` is copied
   from `product.yaml`, with no `trial`, and its `exclusions` hold the disclosure line. `waivers: []`. `decisions`,
   `licenses` and `exclusions` follow [propose-release](#propose-release) step 7.
7. Write `hw/versions/vM/release-proposal.json` with `phase: PROPOSAL`, `test ! -e` first → it validates against the
   `release-manifest` root; written once. Dispatch nothing: `release.inventory` runs in the `run-checks` batch.

### Lite publish-release

Run [publish-release](#publish-release) with its precondition and steps 1 and 3–7. Step 2 copies each planned source
byte for byte to its package path, following no symlink. It then copies these bound extras, each matched to the hash
that binds it, and rehashes every staged file:

| Package path | Source | Role | Bound by |
|---|---|---|---|
| `checks/vM/<check>/<attempt>/**` | every result bundle the readiness verdict accepts | CHECK | the verdict's result and bundle sha256, then the files that bundle manifest lists |
| `record/release/stage-verdict.json` | `hw/versions/vM/stage-verdict.json` | RECORD | the RELEASE decision's `verdict` sha256 |
| `record/release/obligations.yaml` | `hw/versions/vM/obligations.yaml` | RECORD | the verdict's `obligations` sha256 |
| `record/release/release-proposal.json` | `hw/versions/vM/release-proposal.json` | RECORD | the decision's `proposal` sha256 |
| `record/decisions/DR-###.yaml` | the APPROVED RELEASE decision | RECORD | its `hw/decisions/` bytes, which FINAL `approval.file` hashes |
| `record/decisions/statements/DR-###.txt` | that decision's statement | RECORD | the decision's `statement.transcript_sha256` |

A lite package seals only with all three of: the APPROVED RELEASE decision, a PASS readiness verdict and a PASS
`release.integrity`. The step 6 report names the disclosure line as its ceiling row.
