# Release

Owner: cad-release. This file owns revision promotion, the release ceiling, the release proposal, package
completeness, the staged copy, the package integrity check and publication. Gate and waiver decisions are cad-flow
references/gates.md; paths are cad-record `layout.json` (`version.release_proposal`, `release.*`); manifest
fields are the `release-manifest` root; the hash order is cad-record references/record.md "Acyclic hash graph"
step 8. `vF` below is the approved FREEZE version and `vM` the RELEASE attempt.

## Release ceiling

| Release scope service class | A package may be published when | The package is |
|---|---|---|
| `APPEARANCE_CONCEPT` | the RELEASE verdict is PASS | a concept-definition package: form, CAD validity and planned process only; `exclusions` name every functional and physical claim |
| `FIT_ASSEMBLY`, `FUNCTIONAL_DEMO`, `WORKING_PROTOTYPE` | every due `physical.test` is PASS on physical evidence, or each missing test of a permissible non-safety requirement is covered by an approved `UNVALIDATED_TRIAL` waiver | a qualified package; with a trial waiver, a trial package whose `scope.trial` states the waiver DR, envelope, supervision and expiry |
| `PRODUCTION`, `SAFETY_CRITICAL` | additionally, the competent AUTHORITY `product.yaml` names supplied the criteria, allowables and evidence and approves | never by waiver |
| `UNKNOWN` | never | none |

- A safety or regulatory obligation without its physical or authority evidence leaves RELEASE BLOCKED and
  publishes nothing.
- The service class and release scope are the ones GROUND recorded. Another scope is a CHANGE from GROUND
  (cad-flow references/change-impact.md), never an edit at RELEASE.
- A demonstration run ends in a concept-definition package or an approved non-safety trial package, never in
  a functional or physical pass.

## promote-revisions

FREEZE, before `freeze-inputs`; nothing is promoted after FREEZE.

1. Baseline: the record copy in the newest sealed `hw/releases/<REV>/record/`, or none.
2. For every PN in `assembly.yaml`: a part whose `part.yaml`, model and retained sources differ from the
   baseline copy, or that has no baseline, takes the next released letter of `[A-HJ-NPRTUVWY]` after its last
   released letter (`A` when it has none). An unchanged part keeps its released letter.
3. Write the new `rev` into `hw/parts/<PN>/part.yaml` and every occurrence of that PN in `assembly.yaml`.
4. Output: one line per PN `old → new` or `unchanged`, carried into the FREEZE gate packet. A part left on an
   `X` revision blocks FREEZE (`record.semantic`).

## propose-release

RELEASE, after `build-version`; geometry is the FREEZE version's.

1. Confirm the chain: the predecessor FREEZE verdict `vF` is PASS with an APPROVED GATE decision. Confirm
   `product.yaml` `service.class`, `release_scope` and `license` are KNOWN; an UNKNOWN one is a due UNKNOWN for
   `release.inventory`: stop with `DUE_UNKNOWN` and its question.
2. REV: the next letter of `[A-HJ-NPRTUVWY]` unused by every `hw/releases/*`, every `rev` in
   `hw/versions/*/release-proposal.json` and every RELEASE decision; a rejected or failed REV stays consumed.
3. Build guide: fill `<skill-dir>/assets/build-guide.md` from the frozen record into
   `hw/docs/release-<REV>/build-guide.md`; every statement comes from the record, an UNKNOWN prints UNKNOWN,
   a physical test prints PLANNED or WAIVED. License texts the owner approved sit beside it.
4. Planned payload, one entry `{path, source, sha256, bytes, role}` per file, sorted by `path`:

   | Package path | Source | Role |
   |---|---|---|
   | `definition/**` | `hw/versions/vF/definition/**` (GLB is visualization only, as the guide says) | DEFINITION |
   | `manufacturing/<PN>/**` | `hw/versions/vF/manufacturing/<PN>/**`, except `manufacturing.yaml` | MANUFACTURE |
   | `drawings/<SUBJECT>/**` | `hw/drawings/vF/<SUBJECT>/**` | DRAWING |
   | `record/version/**` | `hw/versions/vF/{manifest.json,stage-verdict.json,record/**,source/**}` | RECORD |
   | `record/decisions/**` | every `hw/decisions/DR-###.yaml` and cited proposal the chain GROUND to `vF` names | RECORD |
   | `checks/<vX>/<check>/<attempt>/**` | every result bundle a verdict of that chain lists | CHECK |
   | `bom.yaml`, `bom.csv` | `hw/bom/vF/` | BOM |
   | `verification.yaml`, `verification.csv` | `hw/verification/vF/` | VERIFICATION |
   | `docs/build-guide.md` | `hw/docs/release-<REV>/build-guide.md` | DOCUMENT |
   | `docs/LICENSE-*` | the approved license texts | LICENSE |

   A retained source whose redistribution right is not granted is never copied: it goes to `exclusions` with
   its citation, sha256 and the restriction. Scratch, previews and other notes are never payload.
5. `claims`: from `hw/verification/vF/verification.yaml`, one claim per requirement and outcome, its facets
   grouped, the outcome copied unchanged (PASS, WAIVED, UNKNOWN or PLANNED). A FAIL row blocks the proposal.
6. `scope`: `service_class`, `purpose`, `operating_limits` and `exclusions` copied from `product.yaml`; `trial`
   only when an `UNVALIDATED_TRIAL` waiver is in use, copied from its WAIVER decision.
7. `frozen` (`vF`, its manifest and verdict hashes, its GATE decision), `decisions` (every DR the package
   cites, with hashes), `waivers`, `licenses` (owner-approved values; each third-party item with its source and
   `redistributed`), `exclusions` (every cited item not packaged, with its reason).
8. Write `hw/versions/vM/release-proposal.json` with `phase: PROPOSAL`, `test ! -e` first → it validates
   against the `release-manifest` root; written once.

## release.inventory

The method of family `release.inventory`, check id `release-inventory`, obligation `release.inventory@product`,
`facet: CAD_VALID`, run by a `cad-checker` unit at RELEASE on the proposal and every source it names. One outcome
per criterion:

1. Every planned source exists under `hw/`, outside scratch, and matches its sha256 and bytes.
2. The payload holds what the release ceiling row needs: definition STEP of the assembly and every made part,
   the manufacturing outputs each route names, sheets and coverage for every REQUIRED drawing subject, the BOM
   and verification bundles of `vF`, the record and decisions, every cited check bundle, the build guide and
   the license texts.
3. `claims` equal the verification index; no FAIL row exists; no UNKNOWN, PLANNED or WAIVED row is upgraded.
4. `scope` equals `product.yaml` and the GROUND service class; `trial` is present exactly when an
   `UNVALIDATED_TRIAL` waiver is used, and that waiver is approved, in scope and unexpired.
5. Licenses are owner-approved; a third-party item is copied only with its redistribution right.
6. `exclusions` list every cited item that is not packaged, with its reason.
7. REV is unused; package paths are unique, relative and safe, and none is `release-manifest.json` or under
   `acceptance/`.

The result is PASS only when every outcome is PASS; a missing fact is UNKNOWN `MISSING_INPUT`.

## publish-release

Only after the RELEASE decision is APPROVED and binds `RELEASE_INVENTORY {rev, version: vM, proposal, verdict}`
naming this proposal file and the PASS `vM` readiness verdict, each with its sha256. The post-copy check is not
a second verdict: it never changes the RELEASE verdict or its readiness.

1. Recheck: `test ! -e hw/releases/<REV>`; recompute every planned source sha256. A mismatch stops with
   `HASH_MISMATCH`; a frozen input that must change is a CHANGE and a new version.
2. Stage: `mkdir .cad-work/stage-vM/release-<REV>` (it must not exist; else step 7). Copy each planned source
   byte for byte to its package path, following no symlink, then copy the approved RELEASE decision to
   `record/decisions/`. Rehash every staged file against its expected sha256.
3. Integrity: run `release.integrity` (below) on the staged copy and write its result to
   `acceptance/package-integrity.json` in the staging directory. A result other than PASS publishes nothing.
4. Final manifest: the proposal's fields with `phase: FINAL`, plus `readiness` (the `vM` stage verdict path and
   sha256), `approval {decision, file}`, `inventory` (every staged file, including the decision copy and
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
2. The only other file is the approved decision copy (this result is written after the audit).
3. No symlink, no unsafe path, no file outside the staging directory.
4. `artifacts` list every staged file with its sha256, excluding `acceptance/package-integrity.json` and
   `release-manifest.json`.

The result is PASS only when every outcome is PASS.
