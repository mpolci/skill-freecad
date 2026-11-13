# PCB reference from KiCad

## STEP export

```bash
kicad-cli pcb export step --force --subst-models --user-origin <X>x<Y>mm \
    -o <model>/pcb/board.step <project>.kicad_pcb
```

- `--user-origin` at a board corner (the front-left one, in KiCad page coordinates) makes the
  STEP frame match the enclosure frame: X/Y from that corner, Y pointing to the back
  (KiCad's Y is flipped), **Z = 0 on the board bottom face**. Link the reference with
  `Placement.Base.z = -PcbThickness` to put Z = 0 on the top face.
- On macOS `kicad-cli` is `/Applications/KiCad/KiCad.app/Contents/MacOS/kicad-cli`.
- The STEP is large and regenerable: keep it out of git, keep the generated FreeCAD reference
  document (the link needs it).
- Read the export log: "Could not add 3D model for <ref>" lists the parts without a model
  (common: relays, test points, some QFNs). Add placeholders with the datasheet envelope in the
  reference document, plus anything mounted later (button caps, labels).

## Reading the board

- Board outline, holes, footprint positions: `production/positions.csv` or KiCad's bundled Python
  (`/Applications/KiCad/KiCad.app/Contents/Frameworks/Python.framework/Versions/*/bin/python3`,
  `import pcbnew`; `pcbnew.LoadBoard(path)`, `fp.GetPosition()`, `fp.IsFlipped()`,
  `fp.GetCourtyard(layer).BBox()`, `fp.Pads()`, `pcbnew.ToMM`).
- Courtyard minus its margin gives the body footprint for placeholders.
- Component heights and connector geometry come from the STEP: global bounding boxes per part,
  sections at a few distances from the board edge (sleeve diameters and axes, receptacle shell
  size and centre, how far a connector protrudes past the edge).
- For parts that must clear pads (light pipes around LEDs), compute the pad extents around the
  part centre with `pcbnew`; the STEP has no pads unless exported with `--include-pads`.

## Reference document

Generate it with a GUI-mode script (`ImportGui` keeps colours): import the STEP, put the imported
roots and a `Placeholders` group inside an `App::Part` named `PCB`, save. It is fully generated,
so rebuild it whenever the board changes.
