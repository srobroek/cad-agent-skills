# build123d 0.13.0 calls

Owner: cad-model. Calls checked against build123d 0.13.0 with cadquery-ocp-novtk 8.0.1.1.0 on Python 3.12.13:
signatures read from the installed package and each call run once on 2026-11-04 (source:
[exporters3d.py at v0.13.0](https://github.com/gumyr/build123d/blob/v0.13.0/src/build123d/exporters3d.py); drawing API:
[technical drawing tutorial](https://build123d.readthedocs.io/en/v0.13.0/tech_drawing_tutorial.html)). Environment
pins: references/model-contract.md. Each snippet shows one call on a shape the program already holds; none is a
program. This environment also reads CadQuery STEP files for assembly, readback, previews and drawings.

## Faces and witnesses

Filter by surface type first, then group by position and require exactly one match: `group_by` rounds each face
centre to 6 decimals, and taking `[0]` of a sorted list would pick a tie in kernel order. `position_at` and
`normal_at` normalise u and v over the face's UV bounds.

```python
bores = part.faces().filter_by(GeomType.CYLINDER).group_by(Axis.X)[0]    # cylinders nearest -X
seats = part.faces().filter_by(GeomType.PLANE).group_by(Axis.Z)[-1]      # planes highest in +Z
assert len(bores) == 1 and len(seats) == 1                                # a tie names no feature
point = bores[0].position_at(0.5, 0.5)
normal = bores[0].normal_at(0.5, 0.5)                                     # points out of the material
assert bores[0].distance_to(point) < 1e-6                                 # on the trimmed face, not in a hole
```

## Placement from a record quaternion

`Location` accepts an OCP `gp_Trsf`; `gp_Quaternion` takes w last.

```python
w, x, y, z = occurrence["placement"]["rotation"]
trsf = gp_Trsf()
trsf.SetRotation(gp_Quaternion(x, y, z, w))
trsf.SetTranslationPart(gp_Vec(*occurrence["placement"]["translation"]))
placed = import_step(part_step).moved(Location(trsf))
placed.label = "STD-0002@pin_a"
```

## STEP export and import

A shape's `label` becomes its STEP product name; `timestamp` fixes the header date (pass the attempt's frozen instant,
never the clock). A compound of labelled children exports one product per child.

```python
part.label = "STD-0001"
export_step(part, out / "definition/parts/STD-0001.step", unit=Unit.MM, timestamp=frozen_at)
assembly = Compound(children=placed_occurrences, label="assembly")
export_step(assembly, out / "definition/assembly.step", unit=Unit.MM, timestamp=frozen_at)
```

`import_step` returns a `Solid` for a one-product file and a `Compound` with labelled `children` otherwise; it
replaces spaces in labels with `_`.

```python
back = import_step(out / "definition/assembly.step")
names = [child.label for child in back.children]
ok = back.is_valid and len(back.solids()) == expected_solids
```

## Additive STL at absolute deflection

`export_stl` and `Mesher` mesh relative to edge size; mesh with OCP directly for absolute deflections.

```python
BRepTools.Clean_s(state.wrapped)
BRepMesh_IncrementalMesh(state.wrapped, 0.05, False, 0.1, True)   # 0.05 mm absolute, 0.1 rad, in parallel
writer = StlAPI_Writer()
writer.ASCIIMode = False
writer.Write(state.wrapped, str(out / "manufacturing/STD-0001/part.stl"))
```

## GLB

`export_gltf` writes metres and puts the −90° X rotation (Z-up to Y-up) on the root node; add no other transform.

```python
export_gltf(assembly, str(out / "definition/assembly.glb"), unit=Unit.MM, binary=True,
            linear_deflection=0.05, angular_deflection=0.1)
```

## Flat profiles and previews

`ExportDXF` and `ExportSVG` write 2D shapes in mm; an SVG layer becomes a `<g>` element whose `id` is the layer name.

```python
dxf = ExportDXF(unit=Unit.MM)
dxf.add_layer("profile")
dxf.add_shape(profile_face, layer="profile")
dxf.write(out / "manufacturing/STD-0001/flat.dxf")
visible, hidden = assembly.project_to_viewport((200, -200, 200), viewport_up=(0, 0, 1), look_at=(0, 0, 0))
```

## Text and fonts

`Text` uses the family `"Arial"` unless `font_path` is given; always pass the retained font file.

```python
lettering = Text("STD-0001", font_size=4.0, font_path=str(Path(__file__).parent / "sources" / font_name))
```

| Font file | Bytes | sha256 | License | Where found |
|---|---|---|---|---|
| `ReliefSingleLineCAD-Regular.ttf` | 53,496 | `8b30ea7ea8a2b17fb9d5c70b5c7c37e6a9285b4f8aced4fbd646bc591dba59b3` | OFL-1.1 (`OFL.txt` beside it) | the build123d 0.13.0 package |
| `osifont-lgpl3fe.ttf` | 114,764 | `f681b2419818198a8421b9db01602862367dbc3338c0e8e963fda73f545262b8` | LGPL-3.0, GPL-2.0 or GPL-3.0, with the font exception | FreeCAD 1.1.4 TechDraw (cad-agent v0.1.1) |

Copy the chosen file and its license into `hw/parts/<PN>/sources/`; the input inventory then carries its sha256.
