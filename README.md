# Terrain Maker

A CAD-style terrain editor for low-poly game worlds. Place features from a drawer onto a 2D map: mountains, ranges, cliffs, rivers, lakes, waterfalls and more. Move, rotate, stretch and reroll each one on its own. The editor folds them into one height field, paints the biomes, and lets you fly over it or walk it in first person. Export it as a C header or a binary file.

![Terrain Maker](screenshot.png)

**Live:** https://nhsoule.github.io/Terrain_Maker/

Everything is one `index.html`. There is no build step and nothing to install. The first visit opens the demo island ([`examples/demo.json`](examples/demo.json)). The previous random generator is still at [`legacy.html`](https://nhsoule.github.io/Terrain_Maker/legacy.html), and the git tag `legacy-generator` marks its last version.

## How it works

Nothing is random. The world is **2048 m × 2048 m**, and it is rebuilt from an ordered list of features over a base:

- **Base:** a sea floor (default −20 m) with gentle noise. It can be replaced by an imported map (see Import).
- **Features** apply top to bottom in the feature tree. Each feature has:
  - a type, a name, and its own **seed**. **Reroll** (`R`) changes only that feature's seed, so only it changes.
  - a placement: position, rotation, and scale X / Y / Z (length, width and height in metres).
  - a **blend**:
    - **add** raises the ground by the shape.
    - **carve** lowers it.
    - **max** raises it to Base Z + shape, but never lowers.
    - **flatten** pulls it to Base Z + shape.
  - a **falloff** (m), plus the type's own parameters.
- **Splines:** ranges, cliffs, world edges, valleys, rivers, trails and the shoreline follow a smooth curve through control points.

The rebuild is parametric: change any value and the map follows. A background worker recomputes only the box around what changed. The result is bit-for-bit the same as a full rebuild (the test suite checks this).

## Features (v1)

| Type | What it makes |
|---|---|
| Mountain | A single peak with ridges and gullies running down from the summit. |
| Mountain range | A ridge along a spline with several peaks, saddles and gullies; the ends taper. |
| Hill | A soft rounded hill. |
| Plateau / mesa | A flat top with steep rocky sides. |
| Cliff wall | A wall along a spline, from a low side to a high side (you set the height). |
| World edge | A tall wall (60 m or more) at the edge of the playable area, with a rising slope behind it. |
| River | A channel along a spline. Its bed never climbs downstream, and it ends in the sea or a lake. |
| Lake | A basin that fills to its spill level (the lowest point of its rim). |
| Waterfall | A drop where a river crosses a cliff. Dropping it near one snaps it to the crossing, and it is exported in the features section. |
| Shoreline | A closed spline: land inside, beach along it, sea outside. |
| Valley / canyon | A trough carved along a spline. |
| Trail / ramp | A graded path along a spline, which cuts and fills the ground to an even slope. |
| Terraced hillside | A hill in flat steps. |

**Coming soon** (shown greyed out in the drawer): crater/caldera, dunes, meadow, escarpment, sea stack, arch, overhang, swamp, cove, ravine, crash site, cliff walkway, platform, lookout, and painted biome zones.

**Biomes** are automatic:

- sand near the sea
- rock on steep ground
- snow above the snow line
- forest and plains elsewhere (from noise)
- path on trails

River, lake and sea use one water colour.

## Editing

- **Add:** click a type in the drawer, then click the map. The new feature has a default shape and is selected. `Esc` cancels.
- **Select:** click the feature on the map or in the tree. Its outline and gizmo appear.
- **Move:** drag the feature or its diamond.
- **Rotate:** drag the round handle.
- **Scale and stretch:** the square handles change the length, the width, or both. Height (Z) is in the properties panel.
- **Splines:**
  - Drag the numbered control points.
  - Double-click the curve to insert a point.
  - Right-click a point (or select it and press `Del`) to delete it.
  - The control points are also listed in the properties panel.
- **Snap:** hold `Shift` while dragging to snap to the grid (and to 15° steps when rotating). The grid shown adapts to the zoom.
- **Pan and zoom:** middle-drag, Space + drag, or drag on empty ground pans. The wheel zooms at the cursor, and `F` fits the map.
- **Tree:**
  - Drag a feature to reorder it; ▲ / ▼ also move it.
  - ● / ◌ hides (suppresses) it; `H` toggles the selected one.
  - Double-click to rename.
  - Duplicate with `Ctrl+D` and delete with `Del`.
- **Undo / redo:** `Ctrl+Z` / `Ctrl+Shift+Z` (or `Ctrl+Y`).
- **Preview while dragging:** the outline follows the mouse, and the full-resolution rebuild runs on release.

## Resolution

Resolution is a project setting: **2, 1 or 0.5 m** per sample (1025², 2049² or 4097² samples). The default is **0.5 m**.

You always edit at the chosen resolution, and nothing is reduced while you work:

- **Worker:** only the changed box is rebuilt.
- **2D view:** draws the height field straight from GPU textures.
- **3D view:** reads the same full-resolution textures. Each 64-sample tile picks a level of detail by its distance, and tiles outside the view are skipped. Close ground is drawn at full resolution; far ground uses every 2nd, 4th … sample, since a single screen pixel covers many samples there. The height field itself is not reduced.
- **First person:** stands on the full-resolution field.

Rough numbers, measured in headless Chrome with software rendering, so a normal machine is faster:

| Measurement | Result |
|---|---|
| Full build at 0.5 m | about 7 s (2.7 s at 1 m) |
| Moving a 560 m mountain | 2.2 s |
| Rerolling that mountain | 1.9 s |
| Height + water + biome grids at 0.5 m | 144 MB |

Projects store only the feature list, so the `.json` is a few kB.

## 3D and first person

- **3D:** drag orbits, right-drag (or Shift + drag) pans, and the wheel zooms. The HUD shows the tiles drawn and culled and the level-of-detail mix.
- **Walk:**
  - Click to look around with the mouse (pointer lock).
  - Move with `WASD` or the arrows; `Shift` runs.
  - Ground steeper than 50° blocks you.
  - The **edge fog** toggle hides the end of the world.

## Files

- **Save / Open:** the project as `.json` (format `terrain-maker-project`, version 1). It also autosaves in the browser. Loading a project rebuilds exactly the same heights.
- **Import:** a `.bin` or `.h` exported from Terrain Maker (old or new), used in one of two ways:
  - **Reference:** drawn over the 2D map as an underlay.
  - **Base layer:** replaces the base sea floor, and the features build on top of it.
  The imported grid can be any resolution; it is sampled bilinearly. It is not stored in the project file, so import it again after a reload.
- **Export:** below. Same formats as before.

## Keys

| Key | Action |
|---|---|
| `Ctrl+Z` / `Ctrl+Shift+Z`, `Ctrl+Y` | Undo / redo |
| `Del` / `Backspace` | Delete the selected control point, or the selected feature |
| `Ctrl+D` | Duplicate |
| `R` | Reroll the selected feature |
| `H` | Hide / show the selected feature |
| `F` | Fit the map |
| `Shift` (while dragging) | Snap to the grid / 15° |
| `Esc` | Cancel placing, deselect, close dialogs |
| `Ctrl+S` | Save the project |
| `WASD` / arrows, `Shift` | Walk, run (Walk view) |

## Export

Every export is the full map: one sample per pixel. Pick one of three data formats. Each format also includes a **water surface grid** at the same scale as the height.

| Format | Height + biome | Water surface | Per sample | Height levels | .bin at 2 m (1025²) | 1 m (2049²) | 0.5 m (4097²) |
|---|---|---|---|---|---|---|---|
| `uint16` height + `uint8` biome | 3 B | `uint16` | 5 B | 65536 | 5.0 MB | 20 MB | 80 MB |
| `uint8` height + `uint8` biome | 2 B | `uint8` | 3 B | 256 | 3.0 MB | 12 MB | 48 MB |
| **packed** `uint16` | 2 B | `uint16` (0..4095) | 4 B | 4096 (12 bits) | 4.0 MB | 16 MB | 64 MB |

**Packed:** height in the top 12 bits, biome in the low 4 bits, so `h = v >> 4` (0..4095) and `biome = v & 15`. That allows at most 16 biomes (there are 10). Over a 200 m range one step is about 5 cm.

To get meters, multiply any height or water value by `height_scale`, then add `height_offset`:

```c
world_z = height * height_scale + height_offset
```

- `height_offset` is the Min height. The Export dialog fills Min / Max height from the map's own lowest and highest point (the sea floor is below 0, so it is usually negative).
- `max_height` is the height range above the offset (Max − Min).
- `sea_level` is measured above the offset too.
- `height_scale` is `max_height / 65535`, `/ 255` or `/ 4095` depending on the format.

**Water:** `water[i]` is the water surface in height steps, or 0 where there's no water. A sample is wet when `water[i] > height`, and `depth = (water[i] - height) * height_scale`. River, lake and sea water all count; waterfalls are listed in the features section.

At 8-bit (about 0.9 m steps over a 230 m range), shallow water can round away. Use the 16-bit or packed format if that matters.

### C header (`<name>.h`)

- **Constants:**
  - Map: `name_w`, `name_h`, `name_meters_per_px`, `name_max_height`, `name_height_offset`, `name_height_scale`, `name_sea_level`
  - Chunks: `name_chunk_cells`, `name_chunks_x`, `name_chunks_y`
  - Format: `name_is_packed`, `name_has_water`
- **Shared tables** (behind the `TERRAIN_BIOMES_V4_DEFINED` guard):
  - `enum TerrainBiome`: `BIOME_OCEAN = 0`, `BIOME_LAKE`, `BIOME_RIVER`, `BIOME_POND`, `BIOME_SWAMP`, `BIOME_DESERT`, `BIOME_SNOW`, `BIOME_PLAINS`, `BIOME_ROCK`, `BIOME_FOREST = 9`, `BIOME_COUNT`
  - `terrain_biome_name` and `terrain_biome_is_water`
  - `terrain_biome_rgb` (game colors) and `terrain_biome_key_rgb` (biome PNG key colors)
  - `terrain_biome_water_rgba` (preview water tint)
  - `TERRAIN_KEY_MIN_BRIGHTNESS` (only for decoding old one-image maps)
- **Data:**
  - Separate formats have `name_height[w*h]` (`uint16_t` or `uint8_t`) and `name_biome[w*h]` (`uint8_t`).
  - Packed has `name_packed[w*h]` (`uint16_t`).
  - All formats have `name_water[w*h]`, the same type as the height.
- **Features** (before the data):
  - `enum TerrainLandmark` (behind the `TERRAIN_LANDMARKS_V1_DEFINED` guard): `LANDMARK_CLIFF_WALKWAY = 0`, `LANDMARK_ROCK_ARCH`, `LANDMARK_CRASHED_WRECK`, `LANDMARK_STRANDED_PLATFORM`, `LANDMARK_COUNT`, and `terrain_landmark_name`
  - `name_fall_count` and `float name_falls[N][6]`: `{ x, y, top, bottom, dir, width }`. x and y are world meters, top and bottom are the water surfaces above and below the drop in world z, dir is the downstream direction in degrees counter-clockwise from east (+x), and width is in meters.
  - `name_landmark_count` and `float name_landmarks[M][5]`: `{ type, x, y, z, rot }`. type is a `TerrainLandmark` id, z is the world z of the flat pad, and rot is the facing in degrees counter-clockwise from east.
  - With no entries, the array has one zero row; use the count.

The header is text: about 12 MB at 2 m, 47 MB at 1 m and 186 MB at 0.5 m per sample. Above 2 m most compilers struggle, so use the `.bin` for 1 m and 0.5 m maps. Landmarks are not placed by the editor yet, so `landmark_count` is 0.

### Binary (`<name>.bin`, little-endian, version 2)

| Offset | Type | Field |
|---|---|---|
| 0 | char[4] | `"TMAP"` |
| 4 | u32 | version (2) |
| 8 | u32 | w |
| 12 | u32 | h |
| 16 | u32 | chunk_cells |
| 20 | u32 | chunks_x |
| 24 | u32 | chunks_y |
| 28 | u32 | format: 0 = u16 height + u8 biome, 1 = u8 height + u8 biome, 2 = packed u16 |
| 32 | f32 | meters_per_px |
| 36 | f32 | max_height (height range: Max − Min height) |
| 40 | f32 | height_scale (meters per height or water step) |
| 44 | f32 | sea_level (above Min height) |
| 48 | u32 | biome count (N) |
| 52 | u32 | flags: bit 0 = water grid present, bit 1 = features section present |
| 56 | u32 | data offset (`64 + 32*N`) |
| 60 | u32 | water grid offset |
| 64 | 32 B × N | biome table, see below |
| data offset | | format 0: `u16 height[w*h]` then `u8 biome[w*h]`. Format 1: `u8 height[w*h]` then `u8 biome[w*h]`. Format 2: `u16 packed[w*h]`. |
| water offset | | `u16 water[w*h]` (formats 0 and 2) or `u8 water[w*h]` (format 1) |
| after the water | | features section, if flags bit 1 is set (see below) |

Each biome table entry is 32 bytes:

| Bytes | Field |
|---|---|
| 0 | u8 id |
| 1 | u8 is_water |
| 2-4 | game rgb |
| 5-7 | key rgb |
| 8-11 | water rgba (alpha 0..255) |
| 12-31 | name, NUL padded |

So the file size is `64 + 32*N + w*h*5` for format 0, `64 + 32*N + w*h*3` for format 1, and `64 + 32*N + w*h*4` for format 2, plus the features section if there is one.

The features section starts right after the water grid:

| Bytes | Type | Field |
|---|---|---|
| 0 | char[4] | `"TMFT"` |
| 4 | u32 | version (1) |
| 8 | u32 | waterfall count (F) |
| 12 | u32 | landmark count (L) |
| 16 | 24 B × F | waterfalls: f32 x, y, top, bottom, dir, width |
| then | 20 B × L | landmarks: u32 type, then f32 x, y, z, rot |

These use the same units as the `.h` arrays, except top, bottom and z are above the Min height (like `sea_level`), because the `.bin` has no `height_offset`.

Version 1 files (before water) had no table and the data at 64. Read the offsets from the header rather than hard-coding them.

### Layout

- **Grid:** row-major (`i = row * w + col`), and row 0 is north. World `x = col * mpp`, `y = (h - 1 - row) * mpp`, `z = height * height_scale + height_offset`. Each pixel is a vertex.
- **Cells:** there are `(w-1) × (h-1)` cells. Each cell makes two CCW triangles split on the SW-NE diagonal: SW-SE-NE and SW-NE-NW. A cell's biome is its NW sample's biome.
- **Chunks:** chunk `(cx, cy)` owns cells `col` in `[cx*C, min(cx*C + C, w-1))` and `row` in `[cy*C, min(cy*C + C, h-1))`. It reads samples up to `cx*C + C`, the edge it shares with its neighbor, so chunks never crack.
  - `chunks_x = ceil((w-1) / C)` and `chunks_y = ceil((h-1) / C)`. When `(w-1)` isn't a multiple of C, the last column is `(w-1) - (chunks_x-1)*C` cells wide, and the same goes for rows.
  - With 1025 px and C = 32, every chunk is full.

The comments in the header spell out the exact layout.

The `.bin` header has no `height_offset` field (the layout is unchanged). Add the Min height yourself, or read `height_offset` from the `.h`. When you import a `.bin` back, the Import dialog asks for it.

## Development

`index.html` is the whole app. The heavy lifting is in `terrainCore()`, a pure function that runs in a Web Worker and builds the height, water and biome grids for a rectangle. Each feature type is one entry in its type table, so a new type is one more entry plus a drawer button.
