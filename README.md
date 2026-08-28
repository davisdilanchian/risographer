# Risographer

A browser tool that makes photos look risograph-printed. Drop images into one of
two ink folders, move and rotate them around a page-size canvas, done. No
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

## Using it

1. Click a folder in the sidebar to make it active.
2. Drag images or a `.psd` onto the page (or paste with `Ctrl+V`).
3. Move them around.

| | |
|---|---|
| Move | drag |
| Resize | scroll, or the Size slider |
| Rotate | `Shift`+scroll, `,` / `.` keys (`Shift` = 15°), or the Rotate slider |
| Nudge | arrow keys (`Shift` = 10px) |
| Delete | `Delete` / `Backspace` |
| Switch ink | the "To INK 2" button |

Each folder has its own **ink colour**, **screen angle** and **misregistration
offset**. Keep the two angles at least 30° apart or the overlap will moiré — 15°
and 45° is the default and is what most riso studios use.

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

Drop a `.psd` on the page and it comes in as artwork. It reads the **flattened
composite** that Photoshop writes into every PSD saved with *Maximize
Compatibility* on (the default). RGB and Greyscale, 8-bit, RLE or uncompressed.

It does **not** split a PSD's layers into the two ink folders — that's a much
bigger job, since modern PSDs ZIP their layer data. 16-bit, CMYK, Lab and PSB
files are rejected with a message telling you what to change.

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
- A Letter-size page renders in roughly 50–70 ms, so dragging stays smooth. While
  you drag it samples more coarsely and sharpens up on release.
- `window.__riso` is exposed for scripting and debugging.

## Licence

MIT
