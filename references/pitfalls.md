# FreeCAD pitfalls met in practice

Verified on FreeCAD 26.3 (1.x API). Re-check enumerations on other versions with the probe below.

## Probe the API

```python
import FreeCAD as App
d = App.newDocument("probe")
b = d.addObject("PartDesign::Body", "Body")
for f in b.Origin.OriginFeatures:
    if f.TypeId == "App::Plane":
        print(f.Role, f.Placement.Rotation.multVec(App.Vector(0, 0, 1)))
for t in ("PartDesign::Pad", "PartDesign::Pocket", "PartDesign::LinearPattern"):
    o = b.newObject(t, "F")
    for p in ("Type", "Mode", "SideType"):
        if p in o.PropertiesList and o.getTypeIdOfProperty(p) == "App::PropertyEnumeration":
            print(t, p, o.getEnumerationsOfProperty(p))
```

## Saving and styles

- `freecadcmd` saves only `Document.xml` and shapes: **`GuiDocument.xml` is dropped**, with all
  colours, transparency, display modes, visibility and camera. Use `fcedit.py`.
- When re-injecting it, **archive order matters**. FreeCAD reads the zip sequentially in the
  order files are registered: App files, then `GuiDocument.xml`, then the GUI blobs
  (`ShapeAppearance*`, `LineColorArray*`...) in the order `GuiDocument.xml` references them.
  Writing entries alphabetically silently lost colours.
- Objects created headless have no view provider; on load FreeCAD gives defaults (grey). The GUI
  commands copy the base feature's style to a new feature: `fcedit.py` does the same by cloning
  the base feature's `<ViewProvider>` element (and its blobs under new names).
- Visibility lives in both files; the GUI one wins on load. Patch it in `GuiDocument.xml` for
  objects whose visibility the edit changed (old tip hidden, new tip shown).

## Running scripts

- `freecadcmd file.py` or `-c "exec(...)"`: stdout is frequently invisible; write to a file.
- GUI scripting (`FreeCAD script.py`) starts a separate instance even if FreeCAD is open; end
  with `os._exit(0)` inside `try/except` that prints the traceback, or the window stays open and
  the command hangs.
- `ImportGui.insert` keeps STEP colours; `Import.insert` (headless) does not.
- Screenshots: `Gui.activeDocument().activeView()` then `viewIsometric()` / `viewFront()`,
  `Gui.SendMsgToActiveView("ViewFit")`, `Gui.updateGui()`, `saveImage(path, w, h, "White")`.
  `getCameraNode()` fails without pivy.

## PartDesign behaviour

- Origin planes: **XY** normal +Z; **XZ** sketch x→X, y→Z, normal **−Y** (a positive attachment
  offset moves the sketch toward −Y); **YZ** sketch x→Y, y→Z, normal +X.
- A Pad extrudes along the sketch normal, a Pocket against it; `Reversed` flips both.
- `Midplane` is deprecated: use `SideType = "Symmetric"`.
- **Pocket `TaperAngle` negative narrows with depth** (chamfers, countersinks, drafted recesses).
  Always measure the result.
- `body.newObject("PartDesign::LinearPattern")` **does not move the Tip**: set `body.Tip` yourself,
  or the pattern is silently left out of the body's shape. Pass all `Originals` at creation.
- A pattern based on a feature created in the same edit can fail the first recompute with
  "Cannot transform invalid support shape"; a second recompute fixes it (`fcrun.py` retries).
- **`ThroughAll` pockets cut every feature in their path**, including internal bosses and
  standoffs that later come into range when a parameter changes. Prefer one pocket per wall,
  `Length = wall + margin`, symmetric about the wall mid-plane, and check feature integrity.
- `body.removeObject(f)` relinks the next feature's `BaseFeature`; then `doc.removeObject`.
  Removing and recreating keeps the name free for reuse within the same edit.

## Sketches and expressions

- Bind constraint values with `sketch.setExpression(f"Constraints[{i}]", expr)`; name
  constraints (`renameConstraint`) so later edits can find them by name.
- `DistanceX` / `DistanceY` from the root point (`-1, 1`) accept signed values: negative
  coordinates are fine.
- Rounded rectangle fully constrained: 4 lines + 4 arcs, endpoint-to-endpoint `Tangent` (3
  equations each), 2 Horizontal, 2 Vertical, 3 Equal, 1 Radius, 4 distances = 36 DOF.
- Attachment offset by expression: `sk.setExpression(".AttachmentOffset.Base.z", expr)`.
- `obj.evalExpression("Params.X + 2 mm")` returns a Quantity: handy for initial geometry.

## VarSet

- `vs.addProperty(type, name, group, doc)`; change doc later with `setDocumentationOfProperty`.
- **Never name a parameter like a unit** (`W`, `V`, `A`, `N`, `J`, `K`, `h`, `s`, `m`, `mm`...):
  the expression parser reads `Params.W` as a quantity in watts and fails. Use descriptive names
  (`Width`, `WallThickness`).
- Expression-bound properties are read-only in the GUI: good for derived values.
- Read back from `ExpressionEngine`, expressions inside the VarSet name siblings **without** the
  `Params.` prefix; regexes must accept both forms (`\b((?:Params\.)?)Name\b`).
- Remove a property only after every expression using it has been rebound or deleted.

## Links between documents

- `App::Link` to another document requires the owner document to be saved first ("Owner
  document not saved"): `saveAs`, then add the link, then save.
- The link path is stored relative: run edits on a copy in the same folder, not in /tmp.

## Measuring

- Wall thickness, opening widths: `Part.makeLine(a, b).common(shape).Edges` and read the edge
  bounding boxes along the line.
- Sections: `shape.section(Part.makePlane(...))`.
- Clearances: `shape.distToShape(other)[0]`; overlaps: `a.common(b).Volume`.
- Global placement of imported STEP children: `s = o.Shape.copy(); s.Placement = o.getGlobalPlacement()`.
