# A latent path slip in a sealed attempt does not fail a later attempt

Scenario `sealed-path-slip` · stage `INTENT` · rules: cad-record references/validation.md Attempt scope, Seal check, Path
fields and items 12 and 14; cad-flow references/worker-delivery.md Accept.

## Situation

The GROUND v1 `record.semantic` checker wrote the brief snapshot `installed/cad-record/assets/project-template/hw/product.yaml`
into the `files` of its item table `hw/checks/v1/record-semantic/a1/artifacts/items.json` (items 1 and 14). Nothing caught it,
and the owner approved GROUND v1 (DR-002). The INTENT v2 `record.semantic` checker then applied item 12's path rule to that
immutable v1 table and failed. The cell could not reach INTENT without reopening GROUND (round-7 smoke, Claude full cell, F1;
round-6 Kiro full A/B cell hit the same with `.cad-work/` paths).

## Files

| File | Schema | Role |
|---|---|---|
| [`valid/check-result/sealed-path-slip-ground.json`](../valid/check-result/sealed-path-slip-ground.json) | exit 0 | the GROUND v1 semantic result; its item table, a `schema: null` artifact, holds the latent slip |
| [`valid/decision/sealed-path-slip-gate.yaml`](../valid/decision/sealed-path-slip-gate.yaml) | exit 0 | DR-002 approves GROUND v1 and binds its verdict sha256: v1 is sealed |
| [`valid/check-result/sealed-path-slip-intent.json`](../valid/check-result/sealed-path-slip-intent.json) | exit 0 | INTENT v2 semantic PASS: the v1 result, bundle manifest and item table get the seal check only |
| [`valid/check-result/sealed-path-slip-seal-break.json`](../valid/check-result/sealed-path-slip-seal-break.json) | exit 0 | INTENT v2 semantic FAIL item 12: the v1 item table was edited after DR-002, so its sha256 differs from its binder's |
| [`invalid/check-result/sealed-path-slip-current.json`](../invalid/check-result/sealed-path-slip-current.json) | exit 1 (`expected-errors.yaml`) | a v2 result with the same slip in its own path field |

## Procedure under test

- cad-flow references/worker-delivery.md Accept: the path-field scan rejects a delivery whose result, bundle manifest or
  artifact names a non-`hw/` path, before it can be sealed
- cad-record references/validation.md Path fields: a snapshot or installed template is named only in a message and bound by
  the brief's hashes; item 14's row names the template in its message, never in `files`
- cad-record references/validation.md Attempt scope and Seal check: item 12 judges the path fields of the attempt's own
  results; a sealed attempt's files are only checked against the sha256 their binder records

## Expected

1. Today the v1 checker's delivery is rejected at Accept, naming `$.items[…].files[…]` and the `installed/…` value. The
   corrected delivery names the template in the row message only.
2. A v1 record approved before that rule keeps its table as written. INTENT v2's `record.semantic` runs the seal check from
   DR-002: verdict, result, bundle manifest and item table hash to their recorded sha256s. Item 12 holds and the slip is not
   judged again (`sealed-path-slip-intent.json`).
3. A sealed file whose bytes changed fails item 12 with a finding that names the file and its binder
   (`sealed-path-slip-seal-break.json`). The remedy is a change that reopens GROUND, never an in-place repair.
4. A v2 result with the same slip in its own path field fails: `input_files` takes only `hw/` or `.cad-work/` paths, so the
   schema refuses it (`sealed-path-slip-current.json`). A slip in a `schema: null` artifact fails Accept and item 12.

## Regression fails when

- a later attempt fails item 12, or any item other than the seal check, on a sealed attempt's content
- a sealed file changes and the seal check still passes
- a current result, bundle manifest or artifact that names an `installed/`, `input/`, `lead/` or `.cad-work/` path is
  accepted or passes item 12
