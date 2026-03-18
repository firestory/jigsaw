# Jigsaw puzzle generator

https://draradech.github.io/jigsaw/index.html

A client-side web application that generates jigsaw puzzle cut-line patterns and exports them as SVG files, ready for laser cutting or printing.

## Repository structure

| File | Description |
|------|-------------|
| `index.html` | Landing page linking to both generators |
| `jigsaw.html` | Rectangular jigsaw puzzle generator |
| `jigsaw-hex.html` | Hexagonal / circular jigsaw puzzle generator |

## Functionality

### Rectangular generator (`jigsaw.html`)

Generates a rectangular grid of interlocking jigsaw pieces.

**Parameters**

| Parameter | Default | Description |
|-----------|---------|-------------|
| Seed | random | Random seed that deterministically controls all tab shapes |
| Tab Size | 20 % | Relative size of the interlocking tab on each edge |
| Jitter | 4 % | Amount of random variation applied to tab control points |
| Corner Radius | 2 mm | Radius of the rounded corners of the outer border |
| Tiles | 15 × 10 | Number of puzzle pieces horizontally and vertically |
| Size | 300 × 200 mm | Physical dimensions of the finished puzzle |

**How it works**

- Each internal edge is a cubic Bézier spline with nine control points (`p0`–`p9`): the spline starts and ends at tile corners (`p0`, `p9`) and forms a classic jigsaw-tab bulge through points `p3`–`p6`.
- A seeded sine-based pseudo-random number generator (`random()`) makes results fully reproducible for a given seed value.
- Horizontal lines (`gen_dh`) and vertical lines (`gen_dv`) are generated separately and drawn in different stroke colours (DarkBlue / DarkRed) for easy identification.
- The outer border (`gen_db`) is a rounded rectangle.
- The page embeds a live SVG preview; clicking **Download SVG** exports the same pattern scaled to physical millimetre dimensions.

---

### Hexagonal / circular generator (`jigsaw-hex.html`)

Generates a hexagonally arranged set of interlocking jigsaw pieces that fit inside a circle or hexagon.

**Parameters**

| Parameter | Default | Description |
|-----------|---------|-------------|
| Seed | random | Random seed |
| Tab Size | 27 % | Relative size of the interlocking tab |
| Jitter | 5 % | Random variation on tab control points |
| Diameter | 240 mm | Diameter of the finished puzzle |
| Rings | 6 | Number of concentric hexagonal rings of pieces |
| Circle Warp | off | Warp the hex grid to fit a circle outline |
| Truncate Edge Pieces | off | Replace boundary tabs with straight edges / a circle arc |

**How it works**

- Pieces are arranged on a hex grid addressed by axial coordinates.
- Each internal edge is drawn with the same nine-point cubic Bézier tab used in the rectangular generator.
- Control points are computed in local edge space and then mapped to screen coordinates through a `scale → rotate → warp → translate` pipeline.
- When *Circle Warp* is enabled, each point is projected onto the unit hexagon inscribed in a circle, so that all outer edges lie on a true circular arc.
- `gen_dh` draws the diagonal (angled) edges, `gen_dv` draws the vertical edges, and `gen_db` draws the outer boundary (hexagon or circle).

---

## Output

Both generators export a multi-path SVG file (`jigsaw.svg`) with three separate paths:

- **DarkBlue** – horizontal / angled internal cut lines  
- **DarkRed** – vertical internal cut lines  
- **Black** – outer border  

All paths use `fill="none"` and a thin stroke (`0.1–0.2 mm`), making the file suitable for direct use with laser cutters or vinyl cutters.

## LICENSE info

save function: probably **cc-by-sa**  
(the save function is pieced together based on a bunch of answers of this stackoverflow:
https://stackoverflow.com/questions/19327749/javascript-blob-filename-without-link, that
should make that part cc-by-sa - unless the posters there didn't have the rights to post it)

everything else: whatever you want, **cc0** (https://creativecommons.org/publicdomain/zero/1.0/)

if you want to give credit, a link to this repo is fine, but in no way required
