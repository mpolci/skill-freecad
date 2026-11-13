# Verification

Pick the checks that fit the change: a small tweak needs a valid solid and a measurement of
what changed; moving or reshaping parts calls for the interference and assembly-path checks too.
Report numbers, not impressions. Headless checks should write their report to a file
(`freecadcmd` stdout is unreliable). If the checks grow into a reusable `verify.py` for the
project, ask the user whether to track it in git.

## 1. Bodies

For each printed body: `shape.isValid()`, number of solids (1, or the expected count for
multi-island bodies such as separate light pipes), volume, bounding box. A volume that does not
change after an edit that should change it means the feature is not in the body (check `Tip`).

## 2. Printed parts against each other

`a.common(b).Volume < 1e-3` for every pair. Parts printed together in two materials must touch
without overlapping.

## 3. Interference with reference parts

Load the reference document (or follow the `App::Link`), take every `Part::Feature` with its
global placement plus the link placement, skip pairs whose bounding boxes do not intersect,
report `common().Volume` above tolerance with the part label.

## 4. Assembly path

State how each part goes in (for example the PCB enters the upturned lid along Z). Then:
- inside the inserted part's outline, no material of the receiving part may sit on its path:
  intersect the receiving part with a prism of the **real outline** (with its corner radii;
  a plain bounding box gave false positives in tight corners) extended along the path;
- parts that stick out of the outline (connector sleeves through slots) are swept along the path
  in small steps (0.5 mm) and intersected at each step.

## 5. Feature integrity

Cuts added later can reach features that were fine before. For bosses, standoffs and columns
compute the expected volume (cylinder minus pilot hole) and compare with
`Part.makeCylinder(...).common(body).Volume`; flag "DAMAGED" on mismatch.

## 6. Clearances

For each printed body, the five closest reference parts with `distToShape`; compare with the
intended gaps (button caps, LEDs, connector sleeves). Contacts at 0 must be intentional
(clamping columns, standoffs).

## Visual check

Render a few views from the GUI (isometric, front, rear, a part alone) with `saveImage` and look
at them before reporting: they catch wrong directions and missing features that numbers miss.

## Dimension checks for a specific edit

Measure the thing the edit was about on the test copy: a chamfer's diameter at several depths, a
wall thickness behind a recess, the number of slots on each wall (count gaps along a line), the
position of a moved part. Only then apply the edit to the real file.
