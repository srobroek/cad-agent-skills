# Every disconnected component is checked separately

Scenario `components-separately` · stage `PARTS` · rules: cad-verify references/check-plan.md "Expected scope" steps
2–3. Covers mfg-intent spec §10 item 8.

## Situation

The cable tidy STD-0006 has six solids: a frame, mirrored left and right feet, two identical spacers and a tab the
print operation does not name.

## Files

| File | Schema | Role |
|---|---|---|
| [`valid/part/components-separately-tidy.yaml`](../valid/part/components-separately-tidy.yaml) | exit 0 | the part record: six components, one print operation naming five |
| [`valid/check-result/components-separately-capability.json`](../valid/check-result/components-separately-capability.json) | exit 0 | capability.route result per component; tab UNKNOWN |

## Procedure under test

- cad-verify "Expected scope"
- cad-verify `capability.route`

## Expected

1. Each component gets its own obligation; the mirrored feet stay separate (mirror images are not identical).
2. The identical spacers share one obligation anchored at `spacer_a`, with both listed in `scope.components`.
3. The tab, which no operation names, is `capability.route@STD-0006.tab` UNKNOWN `INCOMPLETE` with a MAJOR finding; it
   is never dropped.
4. The result is UNKNOWN and PARTS is BLOCKED until the route names the tab or the model merges it.

## Regression fails when

- the tab disappears from the scope
- the feet are grouped
- a whole-part obligation stands for its components
