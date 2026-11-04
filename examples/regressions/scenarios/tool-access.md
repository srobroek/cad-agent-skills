# Tool access and operation order

Scenario `tool-access` · stage `DETAIL` · rules: cad-verify references/geometry-motion.md `motion.access`,
`assembly.operations`. Covers mfg-intent spec §10 item 15.

## Situation

Lamp assembly: the hinge screw is tightened after the arm that blocks its hex key is mounted, the cover screwdriver is
named only as "small screwdriver", and the nut pocket is closed by the cover before the nut is inserted.

## Files

| File | Schema | Role |
|---|---|---|
| [`valid/check-result/tool-access-access.json`](../valid/check-result/tool-access-access.json) | exit 0 | access result: hinge FAIL, cover UNKNOWN `MISSING_INPUT` |
| [`valid/check-result/tool-access-operations.json`](../valid/check-result/tool-access-operations.json) | exit 0 | operations result: insert_nut FAIL, closed before install |

## Procedure under test

- cad-verify `motion.access`, `assembly.operations`

## Expected

1. `motion.access@op.fasten_hinge` FAILs in the pre-operation state (arm already installed), not the final pose.
2. `motion.access@op.fit_cover` is UNKNOWN `MISSING_INPUT`: a tool named without dimensions has no envelope.
3. `assembly.operations@op.insert_nut` FAILs: an earlier operation closes the access.
4. The remedies are LAYOUT changes (operation order or an access slot); their findings name LAYOUT as the earliest
   stage.

## Regression fails when

- access is checked only in the final assembled pose
- a named tool is given assumed dimensions
