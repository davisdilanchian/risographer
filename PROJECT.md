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

## v3.2 (2026-08-27) - ink model, pasteboard, invert
- A folder is ONE plate, so its images now composite with "darken" over a white
  ground instead of source-over. Fixes two things Davis reported: white areas
  were ERASING artwork under them on the same plate, and nothing stopped one ink
  doubling up on itself. Different inks still multiply between plates.
  Measured: same-ink overlap 0.469 vs 0.471 alone; white-over-ink keeps 0.469
  where it used to erase to 0; cross-ink [156,145,188] vs predicted
  blue*pink/paper [158,145,188].
- 1.25in pasteboard around the sheet. Off-sheet artwork shows as an un-halftoned
  ghost (clipped to outside the sheet so it never dims real artwork), stays
  clickable/draggable, and the status line counts it. Exports remain page-only,
  verified 2550x3300 with paper in the corner.
- Per-item Invert. Inverted copy built once and cached on the item; folder cache
  key includes the flag. Measured 0.809 -> 0.227 on the subject, 0 -> 0.934 on
  the background.

CAUTION for future sessions: do NOT sample single pixels to check colour in this
app -- it's a halftone, so a single pixel usually lands between dots. An earlier
check "proved" cross-ink multiply was broken when it was fine. Always average
over a region.

## v3.3 (2026-08-27) - ROOT CAUSE: revoked blob URLs

Davis: "the presence of the image hides all the blue or the pink ... even images
beyond its scope". Confirmed from his err.psd export: the BLUE layer was 0.00%
opaque - an entire plate rendered as nothing.

ROOT CAUSE, and it was mine, introduced in the "never fail silently" commit:

    img.onload = () => { addImage(...); URL.revokeObjectURL(url); };

The app keeps that Image for the whole session, but the blob was revoked the
instant it loaded. Browsers may discard decoded image data under memory pressure
and re-decode from src; with the blob revoked there is nothing to decode, so
drawImage silently draws NOTHING. Under the darken model that leaves the density
map pure white -> zero ink -> the whole folder's plate is blank. Explains every
symptom: vanishes entirely, affects images unrelated to the one just added,
intermittent, and worsens as more images are added.

Proven, not guessed: after revoke, fetch(img.src) -> "Failed to fetch" and a
fresh Image from the same src -> onerror. The already-decoded bitmap survives at
first, which is why it works and then dies later.

FIX: keep the object URL for the item's lifetime; release it in a single
removeItem() used by both delete paths. Verified blob stays alive + re-decodable,
both inks render, and deleting does release the URL (no leak).

Note two earlier theories that were WRONG and cost time: (1) that the black
background was simply inking everything - not reproducible; (2) memory pressure
from oversized canvases - his images are all under 1.7MP. The canvas-reuse work
from v3.2 is still worth keeping (render 114ms -> 46ms) but it was not the bug.

## v3.3b - "images disappear at high dot pitch"

Almost certainly the SAME blob-revoke bug as v3.3, not a halftone problem.
Changing dot pitch invalidates every folder's plate cache, which forces a full
re-rasterisation of every image. If a blob was revoked and the browser had
already evicted the decoded bitmap, those images draw nothing. High pitch also
means far heavier renders, so more memory pressure and more eviction -- which is
exactly why cranking the slider triggered it. Awaiting Davis retesting on the
v3.3 build.

While investigating I twice "fixed" the dot radius for canvas antialiasing.
BOTH attempts were wrong and both made it materially worse. DO NOT DO THIS.
Measured by mean luminance, ink is already flat across pitch (spread 0.041 blue,
0.033 pink, 15->80 lpi). The bad attempts faded tone up to 4x at high pitch.

ROOT OF MY ERROR, worth remembering: I measured with a threshold pixel count
(pixels darker than a cutoff). That measures dot VISIBILITY, not ink laid down,
and it rises with pitch purely because small dots overlap more. Always measure
tone as MEAN LUMINANCE over a flat patch. This is the second time a bad metric
sent me after a non-existent bug in this project (the first was single-pixel
sampling for the cross-ink multiply check).

## v3.4 (2026-08-28) - per-image Tone, and Mask (knockout)
- Tone slider (-100..+100) per image: gamma on each channel via a 256-entry LUT,
  baked into the same cached derived copy as Invert. Verified monotonic:
  ink 0.067 -> 0.485 across the range, peak 12% -> 73%.
- Mask makes an element opaque. Needed a GLOBAL stacking order, which did not
  exist -- items only had per-folder array order. Now every plate is rasterised
  by walking ALL items sorted by id (creation order):
    own plate, normal  -> darken       (white is nothing, inks combine)
    own plate, opaque  -> source-over  (replaces, so white knocks out)
    other plate, opaque-> white silhouette (erases that plate's ink)
    other plate, normal-> skipped
  The white silhouette is the image drawn then source-in white, cached per item,
  so knockout follows the artwork's alpha rather than just its rectangle.
- folderKey now includes tone/opaque AND a signature of the OTHER folder's
  opaque items, since those punch holes in this plate. Only opaque ones, so with
  no masks in play the drag cache is exactly as cheap as before (verified 0ms
  fully cached, 89ms one item moved).
- Verified: overlap [158,127,180] overprinting -> [247,158,198] pure pink when
  masked; the mask's white margin knocks blue out to clean paper; blue elsewhere
  untouched; moving a mask invalidates the other folder's cache; exports carry
  both tone and mask.

## v3.5 (2026-08-28) - ROOT CAUSE: a blank plate got CACHED

Davis: "I bring both images in and fit to page and one hides the other ... one
of them disappears and the other remains until I move it or shrink it."

That last clause is the whole diagnosis. Moving or scaling an item is exactly
what changes `folderKey`, so the symptom was never about overlap or Fit page --
it was a stale cache entry that only geometry could clear.

Reproduced in a headless browser: block one frame's drawImage for a single
source (what an evicted decode does), and that folder renders 0% ink AND STAYS
0% on every later frame, recovering only when the item is nudged. Two bugs
compounding:

1. Rasterising straight from the <img>. The v3.3 fix kept the blob alive so the
   browser CAN re-decode, but that re-decode is async -- the drawImage happening
   now still draws nothing. Keeping the blob was necessary, not sufficient.
2. `renderPage` cached that blank plate against the item's geometry, turning a
   one-frame glitch into a permanent disappearance. This is why it looked tied
   to the second image: two full sheets is enough memory pressure to trigger the
   eviction in the first place.

FIXES:
- Every image is adopted into an ImageBitmap we own (`ownPixels`), capped at
  6600px on the long edge -- a full-bleed 11in page at 600dpi, the most any
  export can use. That is memory we hold until close(), not a cache the browser
  may reclaim. Verified: blanking `it.img.src` outright afterwards leaves both
  plates rendering at 0.80 peak ink.
- A plate that rasterises to NO ink while non-opaque artwork is over the sheet
  is never cached, and its sources are re-decoded (`recoverSources`, capped at
  3 tries, reset on success). Verified: the glitch frame reads 0, the very next
  frame reads 0.80 again with nothing touched.
- Masks excluded from that test, since an opaque white element legitimately
  knocks a plate back to bare paper. Verified the v3.4 mask case still caches.

Found while fixing it: `den[]` was never exactly 0 for white. The luma
coefficients sum to 1.0000000000000002, so a white pixel landed on 1.1e-16 --
truthy, so halftone's cheap corner/centre cell reject rejected NOTHING and the
"129ms -> 48ms" fast path from v3.2 had been dead ever since. Anything under
half an 8-bit level is now clamped to 0. Sparse full sheet at 300dpi: 134ms ->
113ms, and ink measured by MEAN LUMINANCE is byte-identical to before across
255/220/180/128/64/0 (0.0536 / 0.1183 / 0.1926 / 0.2765 / 0.3867 / 0.5562), so
the ink model is untouched.

## v3.6 (2026-08-28) - plates that a print shop can actually use, and save/resume

Davis, on the PSD export: "what I wanted was a psd that contained all the images
in their spots in greyscale but without the effects applied, so I can save and
resume progress in another session." And: "let me export the plates as greyscale
images without any of our halftone effects -- this is a tool for outputting
usable templates for risograph printing."

Two different files, so two separate things:

- **Plates** now export CONTINUOUS-TONE GREYSCALE, one PNG per ink, no screen.
  That is what a riso RIP wants; handing it pre-dotted art moires against the
  printer's own screen. Artwork decisions are baked in (tone, invert, mask
  knockouts, the darken model); screen decisions are not (no dots, no angle, and
  no misregistration -- plates must register). The ink limit DOES apply so the
  file matches the preview's coverage.
  Verified on a 255/192/128/0 ramp at 80% limit: plate reads 255/205/153/51,
  R=G=B everywhere, paper margin pure 255, one distinct value over a flat patch
  (vs 250 for the screened version, whose mean ink matches to 0.02).
  The old behaviour is still there as a second button, "Screened".

- **Save project** writes a .psd that is a SAVE FILE, not artwork: each image in
  greyscale at its page rect with nothing applied, plus the session settings.
  Settings live in image resource 4001 (plug-in range, so Photoshop keeps it on
  a re-save); per-image flags live in a tag appended to each layer name,
  `[riso f0 r15 t25 i1 m0]`. Dropping it back in restores the session instead of
  importing layers as new artwork.
  Rotation is stored as a NUMBER, not baked into pixels -- resume keeps the
  slider live and never resamples, at the cost of a rotated image looking
  unrotated if opened in Photoshop. Off-sheet items survive as negative layer
  bounds, which PSD allows and our parser already handled.
  Verified: a session with a rotated + toned item, an inverted item, an off-sheet
  masked item, non-default page/lpi/cover/grain/paper/dpi and edited folder
  colours came back byte-identical in every one of those fields, and the render
  matched to within 0.005 mean ink. psd-tools reads the file independently:
  correct bounds, greyscale layers, layer min 0.259 = the untouched source luma
  (so no invert/tone baked in), and the JSON resource parses.
  A .psd that is NOT ours still adds its layers to the current session and
  leaves the settings alone (verified: 1 item -> 3, lpi and page untouched).

buildPSD grew per-layer rects (it assumed every layer covered the canvas) and an
optional metadata resource. The print PSD export is unchanged -- re-verified with
psd-tools: 4 layers, PAPER/BLUE/PINK/PAPER GRAIN, normal/multiply/multiply/
overlay, grain opacity 31.

CAUGHT IN REVIEW: the v3.5 commit accidentally duplicated a line into docToPSD
(a Python replace hit both matches), which would have thrown ReferenceError on
every PSD export. Fixed here before it shipped.

## v3.6b - rescuing a session from a build with no save button

Davis had work open in the old build and no way to keep it. Two things, so this
never costs anyone their layout again:

- A console snippet (in the README) that reads `window.__riso.S` on the OLD tab
  and downloads `riso-rescue.json`: every image's pixels as a data URL, its
  position/size/rotation/tone/invert/mask and folder, plus page and screen
  settings. Needs nothing from the new build and no reload.
- The new build accepts that JSON on drop, sniffed by contents, and restores it
  through the same applyProject() path the project PSD uses. restoreProject was
  split so both formats share one restore.

VERIFIED end to end against the actual v3.4 build (git show ee38039:index.html):
built a session there with a rotated + toned item, an inverted + masked item, a
custom ink colour, edited angle/offset and non-default page/lpi/cover/grain/
paper, ran the snippet, dropped the resulting 110KB file into the new build:
every field identical.

## v3.7 - multi-select, and moving/scaling as one block

Davis laid a piece out at the wrong paper size and needed the whole arrangement
resized together.

- Drag a box on empty space to select everything it touches; shift+click adds or
  removes one; Ctrl/Cmd+A selects all; Escape clears. Selection is a list of
  item REFERENCES (S.msel), not folder indices, so it survives deletes and
  spans both plates.
- With more than one selected, the panel becomes a group panel: a Scale slider,
  Fit to page, Select all, Deselect, Delete. Dragging any member drags the set,
  the wheel scales it, arrows nudge it, Delete removes it.
- Scaling is about the group's bbox centre and scales POSITIONS as well as
  sizes, so spacing is preserved. The slider measures from the geometry the
  group had when the panel was built and commits on release (so dragging back
  and forth doesn't compound).
- Fit to page: scale the block until it just fits the sheet, then centre it.
  That's the wrong-paper-size fix in one click.
- The bbox is rotation-aware (corners, not the upright rect). Also fixed the
  "off the sheet" counter to use the same test -- it was counting a rotated
  image's upright rectangle, so a correctly fitted layout still warned.

VERIFIED on Davis's own 13-image rescue file: a real mouse drag across the
pasteboard selected all 13; the slider at 150% scaled the bbox 8.84x11.59 ->
13.26x17.38 with every pairwise distance and every width at exactly 1.5x;
switching to A3 and clicking Fit to page produced a bbox exactly 11.69in wide,
centred to 0.000 on both axes, with nothing reported off the sheet.

## Next steps
- Davis to run real photos through it and report how the defaults read
- Layer MASKS are parsed past but not applied - a masked layer imports unmasked

## Future goals
- More than two folders (three-ink riso, 75° screen angle is already reserved)
- Per-folder tonal curve so each plate can carry a different range of the image
- Save/load a layout
- Line and cross screens, not just round dots
