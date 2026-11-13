# Design heuristics for printed PCB enclosures

Examples from one project (a 100 × 100 mm board with connectors on two edges, buttons and LEDs
on the top face, everything else underneath), printed in PETG. They are starting points to
discuss with the user, not rules; the numbers are orders of magnitude.

## Split and assembly

- Deep lid (top + four walls) and a flat bottom plate. The board drops into the upturned lid
  along Z, so **no lid material may sit under the board outline**, and features that must hold
  something sticking out of the board (banana sleeves) are **U-slots open toward the insertion
  direction**, closed by tongues on the plate.
- Print the lid upside down (top face on the bed): slots and vents open upward, no supports.
- Fasten the plate with screws **through the PCB mounting holes**: hollow plate columns, PCB,
  lid standoffs with pilot holes. The board is clamped between the two sets of columns and the
  case needs no room for extra bosses. Seat the screw heads inside the columns, a few mm below
  the board, so short screws (M3 × 10–12 self-tapping) get 6–7 mm of thread.
- Guide the plate with a **rebate** in the wall bottom (inner half of the wall removed, plate lip
  inside it, ~0.1–0.15 mm side clearance). Decide one height datum (columns or wall seat) and
  give the other a gap, or accept zero gap knowingly.
- Gaps between board edge and walls: ~0.2 mm where connectors must be flush; more only where a
  feature needs room (for example a recess that must end before a corner radius). The gaps can
  differ per side: separate parameters, PCB offset rather than a wider case.

## Openings

- Connector openings follow the metal shell (+0.15–0.2 mm per side), and the wall around them is
  **thinned from outside** so the outer face is ~1.2 mm from the receptacle face, room for plug
  overmolds; one recess spanning a row of close ports, one per port when they are far apart.
  Draft the recess sides at 45° (same floor size, larger at the surface): it looks better and
  removes the print overhang.
- Banana / panel sleeves flush with the outer face: wall + gap = sleeve protrusion.

## Buttons and LEDs

- Tactile switches with caps: plain holes (+0.3 mm per side), caps slightly **recessed**
  (~0.3 mm, tolerance stack) with a 45° chamfer on the hole edge against accidental presses.
- Light pipes in clear PETG printed with the lid: **hollow cups open toward the LED, 0.4 mm side
  and top walls**; solid pipes print cloudy. Bring them down to ~0.1 mm above the board so they
  enclose the LED, inside opaque shrouds from the lid against crosstalk. Check pad extents
  against the cup's inner radius.

## Ventilation

- Natural convection in a small closed box is weak: walls carry most of the heat. Estimate:
  PCB → walls ~5–6 K/W, walls → ambient ~3 K/W for ~0.035 m² of black PETG, so a few watts give
  +15–25 K inside; vents remove tens of percent at best. Recommend measuring on the prototype.
- Vertical slots (print without supports), crossing the board level so both the hot layer under
  the board and the space above it are vented, ends at the same distance from the top and bottom
  outer faces; extra slots near the hottest parts (LDO, diodes).
- Cut vents **per wall**, never through-all across the box (see pitfalls).
