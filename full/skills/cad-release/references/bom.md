# Bill of materials

Owner: cad-release. The rows of `hw/bom/vN/bom.yaml` (root `bom`, rows root `bom-row`) are derived from the
record snapshot and the version manifest of one version; they are never authored. `bom.csv` derives from
`bom.yaml` only and has no gate credit. Paths are cad-record `layout.json` (`bom.yaml`, `bom.csv`,
`bom.bundle_manifest`); facts follow cad-record references/record.md "Facts".

## derive-bom

Runs in BOM and FREEZE after `build-version`, so the version manifest exists. The lead derives the rows by
reading the inputs below; no program runs, and the independent `bom.derivation` check re-derives them.

1. Inputs, each recorded with its sha256: `hw/versions/vN/manifest.json`, and from `hw/versions/vN/record/`
   the snapshots of `assembly.yaml`, every `parts/<PN>/part.yaml` and `manufacturing/materials.yaml`, plus
   every published result a mass fact cites (step 5).
2. Rows: walk `assembly.yaml` occurrences from `root` through `parent` links. An occurrence of a part that
   lists `components` gives one row per component (`component` set, `quantity` its `multiplicity`); any other
   occurrence gives one row with `quantity: 1`. `path` is the chain of occurrence ids from the root joined by
   `/`. `unit` is `EA`, except a stock part whose procurement `quantity` is a KNOWN integer in `mm`, `m`, `g` or
   `kg`: that value and unit. Sort rows bytewise by `path`, then `component`.
3. Identity: `PN`, `rev` from the occurrence (it equals part.yaml `rev`, else stop: `record.semantic` defect),
   `title` and `sourcing` from part.yaml; `material_profile` and `route` (ordered route kinds) from the part or
   component intent, or its UNKNOWN fact unchanged.
4. Purchase: for BUY and MADE_TO_DRAWING rows, `purchase.identity`, `specification` and `supplier` from the
   part's procurement; `availability`, `price` and `lead_time` likewise, each KNOWN only with its source and
   date. Anything the record does not state is UNKNOWN with its reason; never 0, never a catalogue guess.
5. Mass, per row, KNOWN only from:
   - a published `mass.properties` outcome for this PN or component on geometry whose manifest digest equals
     this version's: source `kind: CALCULATION`, `reference` the result path and its sha256, `reason` the method;
   - a purchased item's datasheet mass: source `kind: DATASHEET` with its URL or retained artifact.

   Otherwise UNKNOWN: in BOM a made part reads `mass.properties is due at ANALYSIS`.
6. Totals: `part_count` is the sum of `quantity` over `EA` rows. `mass` and `price` are KNOWN only when every
   row's value is KNOWN: the quantity-weighted sum, basis CALCULATED, citing the row sources; otherwise UNKNOWN
   naming the rows that are UNKNOWN.
7. Write `hw/bom/vN/bom.yaml` with `format: cad-agent-skills.bom`, `schema_version: 1`, `version` and
   `version_manifest_sha256` → it validates against the `bom` root.
8. Write `hw/bom/vN/bom.csv` from `bom.yaml` alone (CSV view below).
9. Write `hw/bom/vN/bundle-manifest.json` last: `kind: BOM`, binding `VERSION`, `input` the step-1 files,
   `recipe: {kind: REVIEW, inspected: <the step-1 files>}`, `policy` from preflight, `files` `bom.yaml` and
   `bom.csv` (role `DATA`), `complete: true` → it validates; the verdict lists it in `bundles`.

## CSV view

- UTF-8, LF line endings, a final newline, RFC 4180 quoting when a cell holds a comma, quote or line break.
- Header, in this order: `path,occurrence,PN,rev,component,title,quantity,unit,sourcing,material_profile,route,`
  `purchase_identity,purchase_specification,supplier,availability,mass,mass_units,price,currency,lead_time,`
  `lead_time_units` (one line).
- One line per `bom.yaml` row in the same order. `route` joins route kinds with `+`. A fact cell holds its
  KNOWN value; an UNKNOWN fact holds `UNKNOWN`; an absent optional field is empty.

## bom.derivation

The method of family `bom.derivation`, check id `bom-derivation`, obligation `bom.derivation@product`,
`facet: CAD_VALID`, run by a `cad-checker` unit on the identical inputs; its program is its own, never the
lead's reading.

1. Re-derive every row and the totals from the step-1 inputs with the rules of `derive-bom`.
2. Outcomes, each with the evidence file hashes:
   - rows complete: every occurrence, and every component of a part that lists components, appears exactly
     once, and no row is extra;
   - fields equal: each row's fields equal the re-derived row, in the same order;
   - unknowns kept: no mass, price, availability or lead time is 0, invented or KNOWN without a source;
   - totals: `part_count`, `mass` and `price` follow step 6;
   - CSV: `bom.csv` equals the CSV view of `bom.yaml` byte for byte.
3. A mismatch is FAIL with finding `bom.derivation:product:<occurrence>`; a missing input is UNKNOWN
   `MISSING_INPUT`. The result is PASS only when every outcome is PASS.

NOT Merge occurrences by title, fill a missing mass from density or a supplier family, or price a made part
  from a guess.
