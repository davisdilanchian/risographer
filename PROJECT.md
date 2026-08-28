# Risographer — Project Status

## What we are doing
A browser-based risograph halftone tool. Two ink "folders", a page-size canvas,
drop images in and move them around. The point is zero per-image work: the
halftone screen is anchored to page coordinates, so it applies to whatever is
underneath it regardless of what gets dropped or where it lands.

Live: https://davisdilanchian.github.io/risographer/
Repo: https://github.com/davisdilanchian/risographer (public — required for
Pages on a free account)

## Current progress
v1 shipped and verified live.

## Completed steps
- Single-file vanilla JS app, no dependencies, nothing uploaded anywhere
- Page-anchored halftone: lattice generated in page space from the origin at a
  per-folder angle. Verified by moving an image 37px and confirming 100% of dot
  positions stayed within 1px.
- Per-folder ink colour + screen angle + misregistration offset; multiply
  compositing so the two plates overprint into a third colour
- Dot sizing corrected: radius = cell*sqrt(d/pi) (factor 0.5642), ramping to
  0.707 across the top of the range so solids fill. The first version used 0.72
  flat and over-inked midtones ~60% (0.56 density printed at 0.90 coverage) —
  caught by measuring ink coverage against source density, not by eye.
- Cheap corner/centre reject before the box average; render dropped 129ms -> 48ms
- Move/scale/nudge/delete, move-between-folders, paste support
- Export composite PNG, and per-folder black-on-white plates for a print shop
- Verified end to end on the live Pages URL

## Earlier tracks (superseded, kept in Desktop\RISO)
- `RISO-IZE.jsx` — Photoshop ExtendScript. Dead: Davis no longer has CC, and
  Photopea only partially supports executeAction.
- `FIGMA/` — a self-applying Figma frame (white veil -> tiled dot screen at Color
  Burn -> ink at Screen). Works for ONE ink; two-ink auto-apply was tested and
  fails because the second Color Burn operates on the already-inked composite.
  Also costs a Ctrl+[ per image, which is what motivated building this instead.

## Next steps
- Davis to run real photos through it and report how the defaults read
- Likely tuning: default dot size, and whether the slight dot-gain bias wants
  pulling back

## Future goals
- More than two folders (three-ink riso, 75° screen angle is already reserved)
- Per-folder tonal curve so each plate can carry a different range of the image
- Save/load a layout
- Line and cross screens, not just round dots
