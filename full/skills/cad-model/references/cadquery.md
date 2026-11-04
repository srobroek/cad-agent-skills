# CadQuery 2.8.0 calls

Owner: cad-model. Calls checked against cadquery 2.8.0 with cadquery-ocp 7.9.3.1.1 on Python 3.12.13: signatures read
from the installed package and each call run once on 2026-11-04 (documentation: [import and export at
v2.8.0](https://github.com/CadQuery/cadquery/blob/v2.8.0/doc/importexport.rst)). Environment pins:
references/model-contract.md. Each snippet shows one call on a shape the model already holds; none is a program.

## Faces and witnesses

Select by surface type, sort by position, then take a point on the surface and its normal there. `Face.Center()` of a
cylinder is its centre of mass, which lies on the axis, not on the surface.

```python
bores = sorted(body.faces("%CYLINDER").vals(), key=lambda f: f.Center().toTuple())
point = bores[0].positionAt(0.5, 0.5)    # on the surface
normal = bores[0].normalAt(point)         # points out of the material
seat = body.faces(">Z").val()             # the planar face with the highest +Z
```

## Placement from a record quaternion

`cq.Location` wraps an OCP `gp_Trsf`; `gp_Quaternion` takes w last.

```python
from OCP.gp import gp_Trsf, gp_Quaternion, gp_Vec
w, x, y, z = occurrence["placement"]["rotation"]
trsf = gp_Trsf()
trsf.SetRotation(gp_Quaternion(x, y, z, w))
trsf.SetTranslationPart(gp_Vec(*occurrence["placement"]["translation"]))
location = cq.Location(trsf)
```

## Part STEP

`Assembly.export` with explicit units; the shape becomes a product named by `name` under an unnamed root product.

```python
cq.Assembly(shape, name="STD-0001").export(str(out / "definition/parts/STD-0001.step"), exportType="STEP", unit="MM")
```

## Additive STL

`Shape.exportStl` meshes relative to edge size by default; pass `relative=False` for absolute deflections.

```python
state.exportStl(str(path), tolerance=0.05, angularTolerance=0.1, ascii=False, relative=False)
```

## Text

`Workplane.text` uses the `font` family name unless `fontPath` is given; always pass the retained font file.

```python
font = Path(__file__).parent / "sources" / "osifont-lgpl3fe.ttf"
lettering = cq.Workplane("XY").text("STD-0001", 4.0, 0.6, combine=False, fontPath=str(font))
```

## Not in this environment

- Assembly GLB: `Assembly.export(path, exportType="GLTF")` in 2.8.0 wrote +Y up but millimetre coordinates, not
  glTF metres. The version GLB comes from the build123d environment (references/exports.md).
- Readback, assembly composition, previews and flat profiles run in the build123d environment from the STEP files.
- `cq_warehouse` is not on PyPI; model a fastener head natively from its retained standard.
