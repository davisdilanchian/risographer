# Risographer

A browser tool that makes photos look risograph-printed. Drop images into one of
two ink folders, move them around a page-size canvas, done. No install, no
account, nothing uploaded — it's one HTML file that runs entirely in your browser.

**→ [Open it](https://davisdilanchian.github.io/risographer/)**

---

## The idea

The halftone screen is anchored to **page coordinates**, not to each image.

That's how real riso works: the screen belongs to the plate, not the artwork. So
images slide *through* a fixed dot grid rather than each carrying its own. Two
images in the same folder share one continuous screen, and moving something
doesn't drag its dots along with it.

It also means the effect is genuinely automatic. The screen doesn't care what you
drop in or where you put it — there is nothing to re-apply, ever.

## Using it

1. Click a folder in the sidebar to make it active.
2. Drag images onto the page (or paste with `Ctrl+V`).
3. Move them around.

| | |
|---|---|
| Move | drag |
| Resize | scroll over it, or the Size slider |
| Nudge | arrow keys (`Shift` = 10px) |
| Delete | `Delete` / `Backspace` |
| Switch ink | "Move to…" button in the sidebar |

Each folder has its own **ink colour** and **screen angle**. Keep the two angles
at least 30° apart or the overlap will moiré — 15° and 45° is the default and is
what most riso studios use.

## Printing

**Plates** exports one black-on-white PNG per folder. That's what a riso studio
wants — one file per ink, same canvas size, same positioning. Hand them the two
files and tell them which ink goes on which.

**PNG** exports the composite for screen use.

## Controls worth knowing

| Control | What it does |
|---|---|
| Dot size | Halftone cell in pixels. 4–6 fine, 7 default, 12+ chunky pop-art. |
| Ink limit | Caps maximum coverage. Drop to ~80% if solids look heavy. |
| Offset | Misregistration — nudges a whole plate a few px, like a real press. |
| Grain | Paper texture. |

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

That formula alone never reaches solid, though, because circles on a square
lattice leave corner gaps until the radius hits `cell/√2` = 0.707. So the factor
ramps from 0.5642 up to 0.707 across the top of the range — exact in the
midtones, genuinely solid at full ink.

Measured against source density, coverage now tracks within 0.02–0.05, biased
very slightly heavy. That bias is dot gain, which is what real riso does anyway.

## Notes

- Everything runs locally. No image ever leaves your machine.
- Page renders in roughly 50–70 ms at Letter size, so dragging stays smooth.
  While you drag it samples more coarsely and sharpens up on release.
- `window.__riso` is exposed for scripting/debugging.

## Licence

MIT
