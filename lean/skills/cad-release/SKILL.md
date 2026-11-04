---
name: cad-release
description: Drafts reviewed model-generated drawings, derives the BOM, and proposes and seals release packages. Use for "draw the parts", "derive the BOM", "release REV A".
---

# CAD Release

TRIGGER
+ cad-flow reaches `draft-drawings`, `derive-bom`, `promote-revisions`, `propose-release` or `publish-release`
+ a brief names `drawing.coverage`, `drawing.preview`, `bom.derivation`, `release.inventory` or `release.integrity`
+ a DETAIL part-designer brief asks for drawing intent or drawing sheets
+ "draw the parts", "derive the BOM", "write the build guide", "release REV A", "what does the package claim"
- approve the release inventory, REV, a waiver or a change → cad-flow (references/gates.md)
- build, export or read back geometry → cad-model
- requirement criteria, check plan or verification index → cad-verify

GATES
ASK explicit approval of the exact release proposal and REV (`kind: RELEASE`) before `publish-release` copies a byte

## Workflow

1. Resolve this skill's absolute directory from the brief or preflight (OMP: `skill://cad-release`); assets
   are `<skill-dir>/assets/<file>`, schemas come from cad-record, never from the product directory.
2. Drawing intent, `draft-drawings`, `drawing.coverage` or `drawing.preview`: LOAD references/drawings.md
   (`skill://cad-release/references/drawings.md`) → every REQUIRED subject has sheets and a coverage result
   that validates, or the obligation is UNKNOWN and the stage BLOCKED.
3. `derive-bom` or `bom.derivation`: LOAD references/bom.md (`skill://cad-release/references/bom.md`)
   → `hw/bom/vN/{bom.yaml,bom.csv,bundle-manifest.json}`, each YAML and JSON file conforming to its cad-record prose root.
4. `promote-revisions`, `propose-release`, `publish-release`, `release.inventory` or `release.integrity`:
   LOAD references/release.md (`skill://cad-release/references/release.md`) → a validating proposal, or after
   approval one sealed `hw/releases/<REV>/` and its outer digest reported.
5. Fill a template only from the record: `assets/drawing-intent.yaml` (part `drawing` block),
   `assets/build-guide.md` (package guide), `assets/release-manifest.json` (FINAL manifest shape)
   → no placeholder text left in the written file.

## Rules

MUST Name drawings "reviewed model-generated engineering drawings"; never producer-ready, certified or
  conformant to ISO, ASME or any other standard.
MUST Take every drawn value from the version's STEP; the record supplies requests, criteria and tolerances only.
MUST Record a REQUIRED characteristic the program cannot draw, or a required sheet format or view not
  produced, as UNKNOWN; the stage is then BLOCKED, never passed with a note.
MUST Publish only within the release ceiling in references/release.md; without due physical or authority
  evidence the package is a concept definition or an approved non-safety trial.
MUST Copy claims from the verification index unchanged: UNKNOWN, PLANNED and WAIVED never become PASS.
NOT Edit a frozen input, reuse a REV, overwrite or prune a sealed release, or write the outer digest into it.
NOT Ship or run a helper from this skill; drawing, BOM and copy programs are runtime programs the model
  writes under `.cad-work/`.

OUTPUT
L1 `<procedure or family> vN: PASS|FAIL|BLOCKED — <subject> — <reason>`
   Files — sheets, coverage, BOM, proposal or package, one absolute path per line
   Limits — UNKNOWN, PLANNED, waivers, exclusions, the release ceiling row applied
CAP 150w clean · 300w with blockers
