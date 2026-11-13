---
name: freecad-parametric
description: Create, modify, check and export parametric FreeCAD models (.FCStd). Use it whenever the user works on a FreeCAD model or a 3D-printed part designed in FreeCAD, including enclosures and STL export for slicers.
---

# FreeCAD parametric workflow

The .FCStd belongs to the user as much as to you: they open it, move things, recolour bodies,
tweak parameters. Scripted changes have to start from the file as it is now and leave their work
in place. Everything else here is advice that worked, to be adapted to the project.

## What matters most

- **Modify the existing document, never regenerate it.** A script may create a file once; after
  that, change it where it stands, or the user's manual edits are lost.
- **Keep GUI styles.** `freecadcmd` saves without `GuiDocument.xml` and drops colours,
  transparency and visibility; apply headless edits through `scripts/fcedit.py`, which puts it
  back.
- **Ask the user to save and close the document in FreeCAD before an edit.** FreeCAD keeps no
  lock: a later save from their open copy would overwrite yours.
- **Read the current state first.** Parameters may have changed since your last edit; reason and
  document from what the file contains now.
- **Try non-trivial edits on a copy and measure before touching the real file.** Put the copy
  next to the original so relative links keep resolving.
- **Check the result and say what was not checked.** Scale the check to the change: a valid solid
  and the dimension you touched for a small tweak, interferences and assembly path when parts
  move.
- **Stay simple.** If an approach turns into reverse engineering, stop and offer the simple
  alternative.

## Tools

Find FreeCAD and its version (`/Applications/FreeCAD.app` on macOS:
`Contents/Resources/bin/freecadcmd` headless, `Contents/MacOS/FreeCAD` with GUI) and probe the
API before relying on property names; enumerations change between versions
(`references/pitfalls.md` has a probe snippet). Headless stdout is often lost: write results to a
file and read it.

The scripts in this skill's `scripts/` folder are meant to be used from here:

| Script | Role |
|---|---|
| `fcedit.py` | `python3 <skill>/scripts/fcedit.py [--dry-run] model.FCStd edit.py` runs `edit.py` in `freecadcmd` with `App`, `doc`, `Params` (the VarSet, if any) and `fclib` defined, refuses to save on new recompute errors, restores the GUI styles, gives new PartDesign features the style of their base feature, keeps a backup in `backup/` next to the document |
| `fcrun.py` | Runs inside `freecadcmd` for `fcedit.py` |
| `fclib.py` | Helpers: VarSet properties, sketches on origin planes with an expression offset, constrained circles / rectangles / rounded rectangles / slots with expression dimensions, pad, pocket (taper, symmetric, through all), linear pattern, safe feature removal, rebinding a named constraint |

Copy them into the project only when they need project-specific changes. Write one-off scripts
in the scratchpad. When a script is meant to stay with the project (a custom `verify.py`, an
export or build script, a customised helper), ask the user whether they want it tracked in git.

## Modelling advice

For a new model prefer the Part Design workbench (Body, sketches, pad / pocket / pattern
features): the feature history stays readable and editable by the user in the GUI. When
possible keep existing models in the workbench they were built with, but mix workbenches when it
makes the job simpler. These choices
kept models easy to change;
use them where they help:

- A VarSet (`Params`) with documented properties, `PropertyDistance` for signed values,
  descriptive names (never a unit symbol such as `W` or `V`), computed values as expression-bound
  properties.
- Sketches whose dimensions are expressions, attached to origin planes; one Body per printed part
  or material.
- A coordinate frame that makes parameters readable.
- Reference geometry (a PCB, purchased parts) in its own document, linked with `App::Link`.
- A first build run in GUI mode, so default colours are stored with the file.

Read `references/pitfalls.md` before writing a non-trivial edit: plane normals, taper sign,
pattern tips, through-all pockets and recompute order all caused real failures.

## References

- `references/pitfalls.md` — FreeCAD behaviours that broke edits, with fixes.
- `references/verification.md` — checks and measuring tricks (interference, assembly path,
  feature integrity, clearances, rendered views).
- `references/kicad.md` — PCB reference from a KiCad STEP, placeholders, reading the board.
- `references/printing.md` — STL export for printing, Bambu Studio notes.
- `references/enclosure-design.md` — examples of choices from a printed PCB enclosure, starting
  points rather than rules.
