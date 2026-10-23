# Terrain Maker

A pixel drawing -> heightmap mesh, editor.

You paint a world map as **one color image** in a phone or desktop browser, see it in 3D at full resolution, and export it as a C header (or a binary file) for a game engine.

![Terrain Maker](screenshot.png)

**Live:** https://nhsoule.github.io/Terrain_Maker/

## How the image works

Each pixel is one terrain vertex.

- **Hue = biome.** Each biome has a key color (the swatches). A pixel belongs to the biome whose key color has the closest hue (chromaticity `(r,g,b)/(r+g+b)`), however bright or dark the pixel is.
- **Brightness = height.** A biome's full key color is Max height. Darker shades are lower, down to 30% brightness at 0 m. It never goes black, so dark pixels keep a readable hue.
- The height is read from the channel sum: `height = ((r+g+b) / keySum - 0.3) / 0.7`. That gives about 290 to 535 height steps depending on the biome (0.41 m or finer at 120 m Max height).
- **Sea level** is a map setting. The 3D water plane sits there, and Auto biome colors everything below it Water. The Water biome is wet ground (sea floor, lake or river bed), and its brightness is the bed height like any other biome.

| # | Biome  | Key color (map, max height) | Game color (3D + export) |
|---|--------|------------------------------|---------------------------|
| 0 | Water  | 40, 120, 255                 | 46, 110, 214              |
| 1 | Sand   | 255, 225, 120                | 232, 208, 122             |
| 2 | Grass  | 150, 255, 50                 | 106, 190, 48              |
| 3 | Forest | 40, 255, 120                 | 40, 122, 52               |
| 4 | Dirt   | 255, 150, 70                 | 138, 92, 48               |
| 5 | Rock   | 255, 255, 255 (grey/white)   | 128, 130, 140             |
| 6 | Snow   | 190, 225, 255 (icy)          | 240, 244, 250             |

## Use

- **Raise / Lower / Smooth / Flatten** change brightness and keep the hue.
- **Biome / Fill** change the hue and keep the brightness. **Pick** takes the biome under your finger.
- One finger or left mouse paints. Two fingers or right-drag pans; pinch or the wheel zooms. In 3D: drag to orbit, pinch or wheel to zoom.
- **Generate** makes a seeded island or continent. **Auto biome** colors the map from height and slope.
- **Map** sets the size, max height, sea level, chunk size and vertex size, and shows chunk and mesh stats.
  - **Width and height** in pixels, 16 to 4096 each (they can differ), and **meters per pixel**. Each pixel is a vertex, so the world is `(pixels - 1) * m/px` meters.
  - **Presets:** 1025 × 1025 px @ 2 m = 2048 m, which is exactly 32 × 32 chunks of 32 cells and is the default. There are also 2049 @ 1 m (2048 m), 1001 @ 2 m and 2001 @ 1 m (exactly 2000 m), 1024 @ 2 m (2046 m), 2000 @ 1 m (1999 m), 513 @ 4 m and 256 @ 2 m.
  - A live preview shows the new world size, chunk layout, data size and triangle count before you press **Apply size**. If `(size - 1)` doesn't divide by the chunk size, a hint names the nearest sizes that do. Uneven sizes still work, and the last chunk column or row is just narrower.
  - **Resizing keeps the painting** by *scaling* it (bilinear heights, nearest-pixel biomes) or by *cropping or padding* it 1:1, centered or from the NW corner. New area is sea floor. Undo restores the old size.
- **The 3D preview is full resolution:** one vertex per pixel, the exact grid you export, with no reduced detail. A 2000 × 2000 map is 8 million triangles, which a desktop GPU draws in a few ms. Very big maps can be slow on phones (the 3D corner shows a note and the measured ms per frame).
- **The 2D map is pixel-exact** (nearest-neighbor). Zoom in and a pixel grid appears, and brush size 1 paints exactly one pixel. **F** fits the map, **0** zooms to one screen pixel per map pixel.
- **File**:
  - **Save map PNG** saves the whole map. The settings ride along in a PNG text chunk.
  - **Open** takes a PNG, or an old v1 `.json` project, which is converted.
  - **Save palette PNG** gives you color ramps to use in other paint apps.
  - **Import grey heightmap** replaces the heights and keeps the biomes.
  - Work also autosaves in the browser (IndexedDB, so 4096 × 4096 maps fit), and older autosaves are picked up on first load.
- **Export** writes `<name>.h` or `<name>.bin`, always at full resolution.

You can edit the PNG in any pixel editor. Keep it lossless (no JPEG, no smoothing). Off-palette colors snap to the nearest biome hue when the file is opened.

Keys: Ctrl+Z / Ctrl+Y undo and redo, `[` `]` change brush size, 1-7 pick a biome.

## Export

Every export is the full map: one sample per pixel. Pick one of three data formats:

| Format | Per sample | Height levels | Data at 1025 × 1025 | Data at 2000 × 2000 |
|---|---|---|---|---|
| `uint16` height + `uint8` biome | 3 B | 65536 | 3.0 MB | 11.4 MB |
| `uint8` height + `uint8` biome | 2 B | 256 | 2.0 MB | 7.6 MB |
| **packed** `uint16` | 2 B | 4096 (12 bits) | 2.0 MB | 7.6 MB |

**Packed:** height in the top 12 bits, biome in the low 4 bits, so `h = v >> 4` (0..4095) and `biome = v & 15`. That allows at most 16 biomes (there are 7 now). At 120 m Max height one step is about 3 cm.

Multiply any height value by `height_scale` to get meters. `height_scale` is `max_height / 65535`, `/ 255` or `/ 4095` depending on the format.

### C header (`<name>.h`)

- **Constants:**
  - Map: `name_w`, `name_h`, `name_meters_per_px`, `name_max_height`, `name_height_scale`, `name_sea_level`
  - Chunks: `name_chunk_cells`, `name_chunks_x`, `name_chunks_y`
  - Format: `name_is_packed`
- **Shared tables:** `enum TerrainBiome`, `terrain_biome_rgb` (game colors), `terrain_biome_key_rgb` (map key colors) and `TERRAIN_KEY_MIN_BRIGHTNESS`.
- **Data:** separate formats have `name_height[w*h]` (`uint16_t` or `uint8_t`) and `name_biome[w*h]` (`uint8_t`). Packed has `name_packed[w*h]` (`uint16_t`).

A 2000 × 2000 map is a 20 to 30 MB header. It compiles in a few seconds, but for big maps `.bin` is the easier choice.

### Binary (`<name>.bin`, little-endian)

| Offset | Type | Field |
|---|---|---|
| 0 | char[4] | `"TMAP"` |
| 4 | u32 | version (1) |
| 8 | u32 | w |
| 12 | u32 | h |
| 16 | u32 | chunk_cells |
| 20 | u32 | chunks_x |
| 24 | u32 | chunks_y |
| 28 | u32 | format: 0 = u16 height + u8 biome, 1 = u8 height + u8 biome, 2 = packed u16 |
| 32 | f32 | meters_per_px |
| 36 | f32 | max_height |
| 40 | f32 | height_scale (meters per height step) |
| 44 | f32 | sea_level |
| 48 | u32 | biome count |
| 52 | 12 bytes | reserved (0) |
| 64 | | data: format 0 is `u16 height[w*h]` then `u8 biome[w*h]`, format 1 is `u8 height[w*h]` then `u8 biome[w*h]`, format 2 is `u16 packed[w*h]` |

So the file size is `64 + w*h*3` (format 0) or `64 + w*h*2` (formats 1 and 2).

### Layout

- **Grid:** row-major (`i = row * w + col`), and row 0 is north. World `x = col * mpp`, `y = (h - 1 - row) * mpp`, `z = height * height_scale`. Each pixel is a vertex.
- **Cells:** there are `(w-1) × (h-1)` cells. Each cell makes two CCW triangles split on the SW-NE diagonal: SW-SE-NE and SW-NE-NW. A cell's biome is its NW sample's biome.
- **Chunks:** chunk `(cx, cy)` owns cells `col` in `[cx*C, min(cx*C + C, w-1))` and `row` in `[cy*C, min(cy*C + C, h-1))`. It reads samples up to `cx*C + C`, the edge it shares with its neighbor, so chunks never crack.
  - `chunks_x = ceil((w-1) / C)` and `chunks_y = ceil((h-1) / C)`. When `(w-1)` isn't a multiple of C, the last column is `(w-1) - (chunks_x-1)*C` cells wide, and the same goes for rows.
  - With 1025 px and C = 32, every chunk is full.

The comments in the header spell out the exact layout and how to decode the PNG yourself.

Everything is one `index.html`. There is no build step and nothing to install.
