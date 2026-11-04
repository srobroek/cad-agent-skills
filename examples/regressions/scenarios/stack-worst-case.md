# Nominal clearance passes while the worst-case stack fails

Scenario `stack-worst-case` · stage `DETAIL` · rules: cad-verify references/fits.md `fit.stack`;
references/geometry-motion.md `geometry.clearance`. Covers mfg-intent spec §10 item 14.

## Situation

The lamp slide IF-002 has 0.20 mm nominal clearance between carriage and rail; the stated tolerances reach −0.10 mm at
the worst case.

## Files

| File | Schema | Role |
|---|---|---|
| [`valid/check-result/stack-worst-case-clearance.json`](../valid/check-result/stack-worst-case-clearance.json) | exit 0 | nominal clearance PASS with its nominal-only limitation |
| [`valid/check-result/stack-worst-case-stack.json`](../valid/check-result/stack-worst-case-stack.json) | exit 0 | worst-case stack FAIL: interval −0.10…+0.50 mm |

## Procedure under test

- cad-verify `geometry.clearance`, `fit.stack`

## Expected

1. The clearance result PASSes at nominal and states that tolerance extremes are `fit.stack` evidence.
2. The stack result FAILs: the minimum −0.10 mm misses the ≥ 0 criterion; the finding is MAJOR.
3. DETAIL is FAIL; RSS needs the case's `rss` block and a DR, and would be published beside the worst case, never
   instead of it.

## Regression fails when

- the nominal PASS closes REQ-012
- RSS replaces the worst case without a DR
