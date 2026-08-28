# Risographer

A browser tool that makes photos look risograph-printed. Drop images into one of
two ink folders, move and rotate them around a Letter/A3/custom canvas, done. No
install, no account, nothing uploaded — it's one HTML file that runs entirely in
your browser.

**→ [Open it](https://davisdilanchian.github.io/risographer/)**

---

## The idea

The halftone screen is anchored to **page coordinates**, not to each image.

That's how real riso works: the screen belongs to the plate, not the artwork. So
images slide *through* a fixed dot grid rather than each carrying its own. Two
images in the same folder share one continuous screen, moving something doesn't
drag its dots along, and **rotating a photo turns the photo, not its screen** —
exactly like turning a sheet under a press.

It also means the effect is genuinely automatic. The screen doesn't care what you
drop in or where you put it. There is nothing to re-apply, ever.

## How ink combines

A folder is one **ink plate**, so its images combine the way ink does, not the
way stacked pictures do.

- **White is nothing.** A white background never erases artwork underneath it on
  the same plate. Images composite with `darken` over a white ground rather than
  painting over each other.
- **The same ink doesn't multiply with itself.** Two images overlapping on one
  plate take the denser of the two — one pass of one ink physically can't print
  itself twice. (Measured: 0.471 coverage alone, 0.469 in the overlap.)
- **Different inks do multiply.** Where the two plates cross you get real
  subtractive overprint. (Measured overlap `[156,145,188]` against a predicted
  `blue×pink/paper` of `[158,145,188]`.)

**Invert** flips an individual image. Worth knowing what that means here: since
white is nothing, inverting a photo turns its white background into *solid ink*.
That's usually the point — it's how you get a dense field with the subject
knocked out of it — but keep an eye on the peak-ink readout when you do it.

## Off the sheet

There's a working margin around the page. Anything hanging off it still exists
and is shown as a ghost out there — un-halftoned, because it isn't going to
print — and you can click and drag it back on. The status line counts what's
currently off the sheet. Exports only ever contain the sheet itself.

## Using it

1. Click a folder in the sidebar to make it active.
2. Drag images or a `.psd` onto the page (or paste with `Ctrl+V`).
3. Move them around.

| | |
|---|---|
| Move | drag |
| Resize | scroll, or the Width field (in inches) |
| Rotate | `Shift`+scroll, `,` / `.` keys (`Shift` = 15°), or the Rotate slider |
| Invert | the **Invert** button on the selected image |
| Nudge | arrow keys (`Shift` = 10×) |
| Delete | `Delete` / `Backspace` |
| Switch ink | the "To PINK" / "To BLUE" button |

Each folder has its own **ink colour**, **screen angle** and **misregistration
offset** (in mm). Keep the two angles at least 30° apart or the overlap will
moiré — 45° and 15° is the default and is what most riso studios use.

## Page sizes and units

Letter, A3, A4, A5, square, or **Custom** in inches. The custom fields also take
metric — type `297mm` or `29.7cm` and it converts.

Everything geometric is stored in **inches**, not pixels, so changing the page
size or the export resolution never moves or rescales your layout. Dot pitch is
in **lines per inch** for the same reason: exporting at 600 dpi instead of 300
gives you a sharper file of *the same print*, not different-looking dots. (The
dot count is literally identical at 150, 300 and 600 dpi.)

The on-screen preview caps itself at about 2 megapixels — A3 previews at ~107 dpi
— so dragging stays smooth no matter how big the page is. Exports always use the
full resolution you picked.

## Can you print 100% solid colour?

**No — and you shouldn't try.** This is the single most common riso mistake.

Riso studios advise a maximum of **75–85% coverage per ink**. A true 100% solid
causes visible roller marks, smudging, offsetting onto the next sheet, and paper
jams and tears. [Out of the Blueprint](https://outoftheblueprint.org/files/)
recommends no more than 75% per layer; [Bang Bang
Zine](https://www.bangbangzine.com/pages/riso-print-guide) notes the machine
simply can't lay 100% ink across 100% of the paper and says drop to 75% if you
want a full field. [RISOTTO
Studio](https://risottostudio.com/pages/advanced-print-setup) caps it at 85%.

Two more things worth knowing:

- **Roller marks come from dense ink near the short edge of the paper**, and from
  heavy coverage in the top-centre. Avoid solid blocks there.
- If you really want a big 100% field, leave at least a **½" margin** around it.

The tool defaults the **Ink limit** to 80% and each folder shows a live
**peak ink** readout — green under 85%, amber to 90%, red above. Watch that
number rather than guessing.

Note that ink is translucent, so where the two plates overlap they multiply into
a third colour. Two inks at 80% each is fine; it's per-ink coverage that matters,
though very heavy total coverage will still smudge.

## Importing PSDs

Drop a `.psd` on the page and **each layer comes in as its own draggable image**,
keeping its position relative to the others. Move them apart, put them in
different ink folders, rotate them independently.

- Files are identified by their actual signature, not their name — a PSD that
  arrives with no extension and no MIME type still imports
- Raw, PackBits (RLE) and **ZIP**-compressed layer data — modern Photoshop uses
  ZIP, so this matters
- Layer groups and hidden layers are skipped; adjustment layers (no pixels) too
- Unicode layer names are read where present
- If the PSD records a sane resolution it's placed at true physical size,
  otherwise fitted to the page
- If layer parsing fails for any reason it falls back to the flattened composite,
  so a weird PSD still imports rather than erroring

16-bit, CMYK, Lab and PSB files are rejected with a message telling you what to
change.

Verified against [psd-tools](https://github.com/psd-tools/psd-tools): on fixtures
covering both RLE and ZIP, the parser returns identical layer names, bounds and
pixel values.

## Exporting

| Button | What you get |
|---|---|
| **PNG** | Flat composite for screen. |
| **PSD** | Layered: `PAPER`, one Multiply layer per ink, and `PAPER GRAIN` at Overlay. Opens in Photoshop, Photopea or Affinity. |
| **Plates** | One black-on-white PNG per ink — what a print shop actually wants. Same canvas size, same positioning. |

The PSD writer was verified against [psd-tools](https://github.com/psd-tools/psd-tools):
re-rendering the layer stack and diffing it against the stored composite gives a
maximum difference of **0** across the whole page.

(Pillow's PSD plugin mis-reads these files. That's a known limitation of Pillow's
rudimentary layer support, not a problem with the files — psd-tools reads every
layer, blend mode and ink colour exactly.)

## Riso ink hex values

| Ink | Hex | | Ink | Hex |
|---|---|---|---|---|
| Red | `FF665E` | | Blue | `0078BF` |
| Green | `00A95C` | | Federal Blue | `3D5588` |
| Fluorescent Pink | `FF48B0` | | Purple | `765BA7` |
| Yellow | `FFE800` | | Teal | `00838A` |
| Orange | `FF6C2F` | | Black | `000000` |

## How the dots are sized

Dot **area** has to equal the tone, so radius is `cell × √(d/π)` — factor 0.5642.
Getting this constant wrong is the easy mistake: an earlier version used 0.72 and
over-inked midtones by ~60% (a 0.56-density midtone printed at 0.90 coverage).

That formula alone never reaches solid, because circles on a square lattice leave
corner gaps until the radius hits `cell/√2` = 0.707. So the factor ramps from
0.5642 up to 0.707 across the top of the range — exact in the midtones, genuinely
solid at max ink.

Measured against source density, coverage tracks within 0.02–0.05, biased very
slightly heavy. That bias is dot gain, which is what real riso does anyway.

## Notes

- Everything runs locally. No image ever leaves your machine.
- Each ink folder's plate is cached, so dragging only rebuilds the folder you're
  actually touching. While you drag it also works at a lower resolution, then
  renders at full quality on release. A 9-layer Letter page drags at ~30–60 ms a
  frame and settles in ~130 ms.
- `window.__riso` is exposed for scripting and debugging.

## Licence

MIT
