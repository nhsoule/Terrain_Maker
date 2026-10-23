# Terrain Maker

A terrain editor for low-poly game worlds: generate a base, sculpt it, let it paint the biomes, walk it in first person, and export it as a C header or a binary file.

![Terrain Maker](screenshot.png)

**Live:** https://nhsoule.github.io/Terrain_Maker/

Everything is one `index.html`. There is no build step and nothing to install. It works in desktop and phone browsers.

## Two layers

The map is two separate layers. Each pixel is one terrain vertex.

- **Height**: a float in meters per pixel, between **Min height** and **Max height** (Map settings, default 0 to 200 m, sea level 20 m). The sculpt tools only change this layer.
- **Biome**: one of 10 biomes per pixel (ids 0..9 below). The biome tools only change this layer.

The 2D view shows either layer (**H** toggles):
- **Height**: grey by height with hillshade (sun from the north-west).
- **Biome**: the biome colors over the same relief, with an opacity slider.
- **Contours** (**K**) draws a line every few meters (a round step for about 20 lines over the range), and every 5th line darker.

Hovering shows the height, biome, water depth, slope, pixel and world position.

| Id | Biome | Game color (3D + export) | Water |
|---|---|---|---|
| 0 | Ocean  | 58, 86, 120   | yes |
| 1 | Lake   | 76, 98, 110   | yes |
| 2 | River  | 98, 104, 96   | yes |
| 3 | Pond   | 74, 90, 64    | yes |
| 4 | Swamp  | 78, 92, 52    | yes |
| 5 | Desert | 226, 200, 130 | |
| 6 | Snow   | 240, 244, 250 | |
| 7 | Plains (grassy) | 106, 190, 48 | |
| 8 | Rock   | 128, 130, 140 | |
| 9 | Forest | 40, 122, 52   | |

New biomes will only be added at the end. The packed export format allows up to 16. For water biomes the game color is the bed.

## Workflow

1. **Generate** a base (or keep the island a new map starts with).
2. **Sculpt** the big moves with a large brush: raise or lower whole areas, **Flatten** plateaus and build sites, **Ramp** roads between them, and carve **River** valleys to the sea. Finish with **Smooth**, **Roughen** or **Terrace**.
3. **Auto biome** paints rock on steep slopes, snow up high, beaches, swamps and forest patches. Re-run it after sculpting.
4. Touch up by hand with the **Biome** brush and **Fill**: lakes, ponds, a desert.
5. Walk it with **Player** at your game's eye height and FOV.
6. **Export**.

## Tools

The brush **Size** is in meters, and each tool remembers its own size. There is also **Strength**, and **Falloff** (0 = hard edge, 1 = very soft). A pen's pressure scales the strength. The map shows the brush outline and the falloff ring.

The tools work on the 2D map and on the 3D view. **Sculpt in 3D** (**E**) raycasts the brush onto the terrain. In that mode right drag orbits and middle or Shift drag pans.

**Sculpt (height layer)**

| Tool | Key | What it does |
|---|---|---|
| Raise | R | Pushes the ground up under the brush. |
| Lower | L | Pushes the ground down. |
| Smooth | S | Softens bumps and sharp edges. |
| Flatten | V | Levels to the height where the stroke starts, or to a typed height (tick **fixed**). **Alt+click** (long press on touch) samples the height. |
| Ramp | P | Drag a line to get an evenly graded road or ramp from the start height to the end height. Size is the road width, and Falloff is how far the embankment blends out. |
| Roughen | N | Adds natural bumps. The detail size is in meters. |
| Terrace | T | Turns slopes into flat shelves with steep steps. The step height is in meters. |
| River | C | Drag a path. It carves a U-shaped channel that always runs downhill (high end to low end, at least a 0.1% grade), lowers the banks, and paints the bed River. Size is the channel width, and there is a depth setting. |

**Biome layer**

| Tool | Key | What it does |
|---|---|---|
| Biome brush | B | Paints the selected biome (keys `1`-`9`, `0`). |
| Fill | G | Fills a connected area of one biome. |
| Pick | I | Takes the biome under the cursor. **Alt+click** with the brush or Fill does the same. |
| Select | M | Paints a soft selection (Alt erases). **Generate** can blend into it, and **Auto biome** can be limited to it. |

**Auto biome** sets each pixel from its height, slope and distance to water. The rules, in order:
1. Below sea level: Desert. Under the automatic ocean this is the sea floor.
2. Slope ≥ the rock angle (36°): Rock.
3. Above the snow line: Snow. By default the snow line is 80% of the way from the sea to the highest peak, with a ragged edge.
4. Near water and just above it: Desert (beach).
5. Flat, low and near water: Swamp, in patches.
6. Everything else: Forest patches (thinning with altitude) over Plains.

You can change each threshold and the forest and swamp amounts. The scope is the whole map, the selection, or only pixels you haven't painted by hand. Hand-painted water can be kept.

## Generator

**Generate** takes a **seed** and one of four presets. It always makes the same map from the same seed and settings, on any device.

| Preset | Look |
|---|---|
| Island | Sea all round, mountain ranges inland (the default map is Island, seed 7) |
| Continent | Mostly land, with bays and inland lakes |
| Mountains & valleys | Ridged ranges with deep valleys |
| Rolling hills | Gentle, open land |

How it builds a map:
1. Domain-warped fBm noise makes the big landforms.
2. Ridged noise, masked to mountain areas, makes the ranges.
3. Hydraulic erosion cuts gullies and settles the slopes.

**Feature size**, **Mountains**, **Warp**, **Erosion**, **Edges** (island falloff or none) and optional **Terraces** can be tuned. It runs in a Web Worker with a progress bar, so the page stays responsive. It takes about 2 s at 1025 × 1025 on a desktop.

**Only in selection** blends the new terrain into the selected area. **Auto biome after** paints the result. Undo brings the old map back.

## Water

| Water | How it works |
|---|---|
| Ocean | Automatic: ground below **Sea level** that connects to the map edge (or to painted Ocean) is sea, and its surface is at sea level. Sculpt the coast and the shoreline follows. Turn it off with **Auto ocean** in Map settings to paint Ocean by hand. |
| Lake, Pond | Paint the biome in a dip. Each connected patch fills to its **spill level**: the lowest ground around its edge, and at least 0.3 m above its lowest bed. A bed above that surface is dry shore. |
| River | The surface is the bed + 0.8 m, and it flows downhill. A priority flood runs upstream from the river's outlets (where it touches Ocean, Lake or Pond). The surface never drops going upstream, so a dip in the bed becomes a pool. The River tool carves the bed for you. |
| Swamp | Wet ground with a 0.25 m film of standing water. |

A sample is under water where `surface > bed`, and `depth = surface - bed`. The 3D view draws the surfaces see-through, tinted per water type.

## Files

**File** menu:
- **Save project** (`.tmproj`): both layers at full float precision plus all the settings. It is a compressed binary file.
- **Open project / old map** takes:
  - a `.tmproj`
  - a height or biome PNG
  - a plain greyscale heightmap
  - an older Terrain Maker map: a one-image PNG or a `.json` project. These are converted (see below).
- **Save height PNG**: a 16-bit greyscale PNG, black = Min height, white = Max height. The range is stored in the PNG, so it opens at the same heights (within 1/65535 of the range).
- **Save biome PNG**: an RGB PNG, one key color per biome (the swatch underline colors).
- **Import height PNG / Import biome PNG** replace one layer and keep the other. Images of another size are scaled: bilinear for height, nearest pixel for biomes. Any 8- or 16-bit grey or color PNG works for height, using its luminance.
- **New flat map** starts a flat map.

Work autosaves in the browser (IndexedDB, so 4096 × 4096 maps fit).

**Older maps** (one color image, where hue = biome and brightness = height) are split into the two layers when you open them, including old autosaves. Heights keep their values (Min height becomes 0). Biomes are decoded with the palette the map was saved with:
- Version 3 Fall becomes Forest.
- Version 2 Water becomes Ocean, Sand and Dirt become Desert, and Grass becomes Plains.

## Map settings

- **Width and height** in pixels (16 to 4096 each, they can differ) and **meters per pixel**. Each pixel is a vertex, so the world is `(pixels - 1) * m/px` meters.
  - The default is 1025 × 1025 px @ 2 m = 2048 m, which is exactly 32 × 32 chunks of 32 cells.
  - Presets: 2049 @ 1 m, 1001 @ 2 m, 2001 @ 1 m, 1024 @ 2 m, 2000 @ 1 m, 513 @ 4 m and 256 @ 2 m.
  - Resizing keeps the map by scaling it or by cropping or padding it 1:1. Undo restores the old size.
- **Min / Max height**: the range the export spreads its height steps over. Keep it tight for finer steps.
- **Sea level**, **Auto ocean**, chunk size, vertex size, water preview.
- First person and edge fog settings (below).

**The 3D preview is full resolution**: one vertex per pixel, the exact grid you export, with no reduced detail. **The 2D map is pixel-exact**. **F** fits it, and **=** zooms to one screen pixel per map pixel.

## Keys

| Keys | Action |
|---|---|
| R L S V P N T C | Raise, Lower, Smooth, Flatten, Ramp, Roughen, Terrace, River |
| B G I M | Biome brush, Fill, Pick, Select |
| `1`-`9`, `0` | Pick a biome (`1` = Ocean ... `9` = Rock, `0` = Forest) |
| `[` `]` | Brush size |
| `,` `.` | Strength |
| Alt + click | Flatten: sample the height. Biome tools: pick. Select: erase. (On touch, long press.) |
| H | Height or biome view |
| K | Contours |
| E | Sculpt in 3D |
| F, `=` | Fit the map, 1:1 zoom |
| Ctrl+Z, Ctrl+Y (Ctrl+Shift+Z) | Undo, redo |
| Esc | Cancel the stroke, close a dialog, leave first person |
| W A S D, Shift, Space | First person: move, sprint, jump |

Mouse and touch:
- **2D map:** the left mouse or one finger uses the tool. Right or middle drag pans and the wheel zooms. Two fingers pan and pinch-zoom.
- **3D view:** drag orbits, right drag pans and the wheel zooms. Two fingers pan and zoom.

## First person (Player)

Walk the map at your game's player scale, to judge whether it is big enough.

![First person view](screenshot_fp.png)

1. Press **Player**, then click or tap a spot on the 2D map or in the 3D view. The 3D view becomes the player's eyes, at full resolution.
2. Look around and walk:
   - **Desktop:** click the 3D view to capture the mouse for mouse-look (dragging also works). `WASD` or the arrows move, `Shift` sprints, `Space` jumps.
   - **Touch:** your left thumb is a joystick (push it all the way to sprint). Drag on the right half to look. There is a **Jump** button.
3. **Esc** or the **Exit first person** button (top right) goes back to the orbit view. **Player** again lets you pick a new spot.

While you walk:
- **The readout** (top left) shows your position in meters and pixels, the chunk you are in, the ground height and eye height, the biome, any water depth, the distance to the nearest map edge and which edge it is, your compass facing, and the vertical and horizontal FOV.
- **On the 2D map,** a red dot with your view cone follows you, as it does on the small map in the corner. You can keep sculpting on the 2D map, and the view updates live.
- **Chunks on** draws chunk lines on the ground.
- **Movement:**
  - Your feet follow the exact exported mesh, using the same two triangles per cell.
  - The map edge is a wall.
  - Walking off a ledge drops you with gravity.
- **Water:**
  - You wade until the water is deeper than eye height − 0.3 m.
  - Past that, you swim with your eyes 0.3 m above the surface, at half speed.

**Map settings → First person** (saved with the project):

| Setting | Default | Notes |
|---|---|---|
| Eye height | 1.7 m | above the feet |
| Vertical FOV | 70° | the horizontal equivalent is shown for the 3D view's current shape, 16:9 and 4:3 (70° v = 102° h at 16:9) |
| Walk speed | 5 m/s | |
| Sprint multiplier | 2× | Shift, or a fully pushed joystick |
| Near clip | 0.1 m | |
| Far clip | 0 = auto | auto is the map's corner-to-corner distance (3094 m for 2048 m), so the whole map is always in view |

## Edge fog

The **Edge fog** button in the 3D view (also in **Map settings**, off by default) fades the terrain and water into a light haze toward the map boundary. That way you don't see a hard edge where the map ends.
- The fog grows over the last **Fog width** meters (150 m by default), based on each point's distance to the nearest map edge.
- It stays a little lighter close to you.
- Beyond the edge, and below the horizon, there is a haze backdrop instead of empty sky.
- It works in the orbit view and in first person.
- It is preview only. It is not exported.

## Export

Every export is the full map: one sample per pixel. Pick one of three data formats. Each format also includes a **water surface grid** at the same scale as the height.

| Format | Height + biome | Water surface | Per sample | Height levels | Data at 1025 × 1025 | Data at 2000 × 2000 |
|---|---|---|---|---|---|---|
| `uint16` height + `uint8` biome | 3 B | `uint16` | 5 B | 65536 | 5.0 MB | 19.1 MB |
| `uint8` height + `uint8` biome | 2 B | `uint8` | 3 B | 256 | 3.0 MB | 11.4 MB |
| **packed** `uint16` | 2 B | `uint16` (0..4095) | 4 B | 4096 (12 bits) | 4.0 MB | 15.3 MB |

**Packed:** height in the top 12 bits, biome in the low 4 bits, so `h = v >> 4` (0..4095) and `biome = v & 15`. That allows at most 16 biomes (there are 10). With the default 0 to 200 m range one step is about 5 cm.

To get meters, multiply any height or water value by `height_scale`, then add `height_offset`:

```c
world_z = height * height_scale + height_offset
```

- `height_offset` is the Min height. It is 0 unless you set a Min height below 0.
- `max_height` is the height range above the offset (Max − Min).
- `sea_level` is measured above the offset too.
- `height_scale` is `max_height / 65535`, `/ 255` or `/ 4095` depending on the format.

**Water:** `water[i]` is the water surface in height steps, or 0 where there's no water. A sample is wet when `water[i] > height`, and `depth = (water[i] - height) * height_scale`. Only water biomes are ever wet.

At 8-bit (0.78 m steps over 0 to 200 m), shallow water can round away: the 0.25 m swamp film, and in places the 0.8 m river. Use the 16-bit or packed format if that matters.

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

A 2000 × 2000 map makes a header of about 30 to 47 MB. It compiles, but for big maps `.bin` is the easier choice.

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
| 52 | u32 | flags: bit 0 = water grid present |
| 56 | u32 | data offset (`64 + 32*N`) |
| 60 | u32 | water grid offset |
| 64 | 32 B × N | biome table, see below |
| data offset | | format 0: `u16 height[w*h]` then `u8 biome[w*h]`. Format 1: `u8 height[w*h]` then `u8 biome[w*h]`. Format 2: `u16 packed[w*h]`. |
| water offset | | `u16 water[w*h]` (formats 0 and 2) or `u8 water[w*h]` (format 1) |

Each biome table entry is 32 bytes:

| Bytes | Field |
|---|---|
| 0 | u8 id |
| 1 | u8 is_water |
| 2-4 | game rgb |
| 5-7 | key rgb |
| 8-11 | water rgba (alpha 0..255) |
| 12-31 | name, NUL padded |

So the file size is `64 + 32*N + w*h*5` for format 0, `64 + 32*N + w*h*3` for format 1, and `64 + 32*N + w*h*4` for format 2.

Version 1 files (before water) had no table and the data at 64. Read the offsets from the header rather than hard-coding them.

### Layout

- **Grid:** row-major (`i = row * w + col`), and row 0 is north. World `x = col * mpp`, `y = (h - 1 - row) * mpp`, `z = height * height_scale + height_offset`. Each pixel is a vertex.
- **Cells:** there are `(w-1) × (h-1)` cells. Each cell makes two CCW triangles split on the SW-NE diagonal: SW-SE-NE and SW-NE-NW. A cell's biome is its NW sample's biome.
- **Chunks:** chunk `(cx, cy)` owns cells `col` in `[cx*C, min(cx*C + C, w-1))` and `row` in `[cy*C, min(cy*C + C, h-1))`. It reads samples up to `cx*C + C`, the edge it shares with its neighbor, so chunks never crack.
  - `chunks_x = ceil((w-1) / C)` and `chunks_y = ceil((h-1) / C)`. When `(w-1)` isn't a multiple of C, the last column is `(w-1) - (chunks_x-1)*C` cells wide, and the same goes for rows.
  - With 1025 px and C = 32, every chunk is full.

The comments in the header spell out the exact layout.

The `.bin` header has no `height_offset` field (the layout is unchanged). If you set a Min height below 0, add it yourself, or read `height_offset` from the `.h`. The Export dialog reminds you when it isn't 0.

Everything is one `index.html`. There is no build step and nothing to install.
