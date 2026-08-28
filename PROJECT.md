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

## v2 (2026-08-27, same day)
- Rotation: per-item `rot`, applied when rasterising into the density map so the
  screen stays page-locked. Hit-testing inverse-rotates. Shift+wheel, , / . keys,
  or the slider. Verified: rotating 90 deg moved the test bar out of the sampled
  region and hit-testing at +290px flipped false -> true.
- Ink limit default lowered 88 -> 80, plus a live per-folder "peak ink" readout
  (green <85, amber <=90, red above). Riso studios cap at 75-85% per ink; a true
  100% solid causes roller marks, smudging and jams. Answered in the README with
  sources.
- PSD export: hand-written PSD writer (PackBits RLE, big-endian). Layers are
  PAPER (normal), one Multiply layer per ink, PAPER GRAIN (overlay, opacity from
  the grain slider), plus the merged composite.
  VERIFIED with psd-tools (independent lib): all 4 layers, exact blend modes,
  exact ink RGB, and re-rendering the layer stack vs the stored composite gives
  max abs difference 0 across the page. Pillow mis-reads it -- that's Pillow's
  rudimentary PSD layer support, confirmed by hand-decoding the channel data.
- PSD import: parses the flattened composite. RGB + Greyscale, 8-bit, RLE + raw
  both verified against hand-built fixture PSDs; 16-bit / CMYK / PSB rejected
  with clear messages. Does NOT split layers into folders (modern PSDs ZIP layer
  data -- would need an inflate path).

## v3 (2026-08-27, same day)
- Inks swapped to Davis's: BLUE #0177bd @45deg, PINK #ff48b0 @15deg. Folders
  renamed to match, so plate exports are plate-1-BLUE.png etc.
- ALL geometry moved to INCHES (was page pixels). Page size and export dpi are
  now pure view settings -- changing either never moves or rescales the layout.
  Verified: switching Letter -> A3 leaves every item's x/y/w byte-identical.
- Dot pitch is now LPI, not px. Verified resolution-independent: dot count is
  identical (102) at 150 / 300 / 600 dpi.
- Page presets Letter/A4/A3/A5/Square + Custom in inches; custom fields also
  parse "297mm" and "29.7cm".
- Export dpi selector 150/300/600 with a live px + megapixel readout that warns
  above 18MP and 40MP. Preview auto-caps at ~2.2MP (A3 previews at ~107 dpi) so
  dragging stays smooth; exports use full resolution. Busy overlay on export.
- PSD import now brings in EACH LAYER as its own draggable item, preserving
  relative position. Handles raw / PackBits / ZIP (via DecompressionStream) and
  ZIP-with-prediction; skips groups, hidden and adjustment layers; reads luni
  unicode names and resource 1005 for true physical placement; falls back to the
  flattened composite if layer parsing throws.
  VERIFIED against psd-tools on RLE + ZIP fixtures: identical names, bounds and
  pixel means.
- PSD export now writes resource 1005 so Photoshop opens it at the right size.

## v3.1 (2026-08-27) - real-world PSD bug
Davis dragged in Downloads\psd and nothing happened. Two bugs:
1. PSD detection was by filename + MIME. His file was literally named "psd"
   with NO extension and an empty MIME type, so it matched neither branch.
   Now sniffs the 8BPS magic bytes instead.
2. WORSE: any dropped file matching neither branch was silently discarded - no
   error at all, which reads as "the app is broken". Now every unusable drop
   reports what it was and what's supported, and decode errors surface too.

The file itself was a valid 792x1224 8-bit RGB PSD. Parser handles it: all 9
pixel layers in 95 ms, bounds matching psd-tools exactly (including one layer
extending past the canvas at x=444 w=395, another at y=-1); adjustment and
empty type layers correctly skipped.

Also fixed the performance this exposed: 9 layers took 193 ms/frame because
every render rebuilt BOTH folders. Added a per-folder plate cache keyed on that
folder's own geometry + screen settings (items carry a stable id), and dropped
preview resolution ~3.5x while dragging since density rasterisation is the real
cost and `fast` sampling doesn't touch it. Drag is now 30-60 ms/frame.

## Next steps
- Davis to run real photos through it and report how the defaults read
- Layer MASKS are parsed past but not applied - a masked layer imports unmasked

## Future goals
- More than two folders (three-ink riso, 75° screen angle is already reserved)
- Per-folder tonal curve so each plate can carry a different range of the image
- Save/load a layout
- Line and cross screens, not just round dots
