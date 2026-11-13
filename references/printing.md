# Exporting for printing and slicer notes

## STL export

- One STL per printed body, exported by a script that only reads the document.
- Orient for printing in the export (for example a lid flipped so its top face is on the bed),
  centre on X/Y = 0, Z = 0 on the bed.
- Bodies of one multi-material object (lid + light pipes) get **the same transform**, computed
  from the main body, so they stay aligned when imported together.
- Mesh: `MeshPart.meshFromShape(Shape=s, LinearDeflection=0.01, AngularDeflection=0.0873,
  Relative=False)`; refuse to write unless `mesh.isSolid()` and not `mesh.hasNonManifolds()`.
- Print the facet count and bounding box of each file in the report.

## Slicer project: let the user build it

Import both STL files of a multi-material object together and accept "load as a single object
with multiple parts", then assign the filament per part. Generating a Bambu Studio 3MF from
scratch was a dead end: the CLI does not resolve preset inheritance, the project needs nozzle and
AMS keys the GUI writes, and the resulting file hung the GUI.

## Editing a Bambu Studio 3MF the user saved

Minimal, surgical edits are safe: back the file up, change only
`Metadata/project_settings.config` (JSON) and `Metadata/model_settings.config` (XML), keep the
other zip entries byte-identical and in the same order, diff against the backup.

- Filament settings are flattened per extruder variant: `filament_extruder_variant` and
  `filament_self_index` say which entry belongs to which filament and variant. User presets use
  `"nil"` for "as the parent" and may list fewer variants than the printer.
- Dual-extruder printers (X2D, H2D): `filament_map_mode = "Manual"` plus `filament_map`
  (filament → extruder) globally **and** `filament_map_mode` / `filament_maps` in each `<plate>`,
  otherwise the plate's "Auto For Flush" wins. Check each extruder's printable area
  (`extruder_printable_area`) against the part positions.
- `curr_bed_type` selects the plate (PETG: textured PEI releases well and textures the face on it).

## "Reload from disk" shifts multi-part objects

After a part's STL changes size, Bambu Studio re-centres the new mesh on its own bounding box
but keeps the volume offset computed for the old mesh: the part moves relative to the others and
the object may be lifted off the bed. Fix in the 3MF, for the changed part:
- new centre `c` = bounding-box centre of the new STL; object origin offset `o` = the main
  part's `source_offset`;
- part `matrix` translation and the matching `<component transform>` (and the `assemble_item`
  for that volume) = `c - o`; `source_offset_*` = `c`;
- build `<item>` Z back to the value that puts the object on the bed;
- check by computing every part's world Z range from the stored vertices.
Alternatively the user re-imports the STL files together as one multi-part object.
