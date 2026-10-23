# Terrain Maker

A pixel drawing -> heightmap mesh, editor.

You paint a world map as **one color image** in a phone or desktop browser, see it in 3D, and export it as a C header for a game engine.

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
- **Map** sets size, meters per pixel, max height, sea level, chunk size and vertex size, and shows chunk and mesh stats.
- **File**:
  - **Save map PNG** saves the whole map. The settings ride along in a PNG text chunk.
  - **Open** takes a PNG, or an old v1 `.json` project, which is converted.
  - **Save palette PNG** gives you color ramps to use in other paint apps.
  - **Import grey heightmap** replaces the heights and keeps the biomes.
  - Work also autosaves in the browser, and an old v1 autosave is converted on first load.
- **Export** writes `<name>.h`.

You can edit the PNG in any pixel editor. Keep it lossless (no JPEG, no smoothing). Off-palette colors snap to the nearest biome hue when the file is opened.

Keys: Ctrl+Z / Ctrl+Y undo and redo, `[` `]` change brush size, 1-7 pick a biome.

## Export format (`<name>.h`)

- **Constants:** `name_w`, `name_h`, `name_meters_per_px`, `name_max_height`, `name_height_scale`, `name_sea_level`, `name_chunk_cells`, `name_chunks_x`, `name_chunks_y`.
- **`enum TerrainBiome`**, `terrain_biome_rgb` (game colors), `terrain_biome_key_rgb` (map key colors) and `TERRAIN_KEY_MIN_BRIGHTNESS`.
- **`name_height[w*h]`:** `uint16_t`, or `uint8_t` if you pick 8-bit. Multiply by `name_height_scale` to get meters.
- **`name_biome[w*h]`:** `uint8_t`.

Both grids are decoded from the image. Data is row-major and row 0 is north. Each pixel is a vertex and each cell makes two CCW triangles split on the SW-NE diagonal. A cell's biome comes from its NW pixel. Chunks share edge samples, so they don't crack. The comments in the header spell out the exact layout and how to decode the PNG yourself.

Everything is one `index.html`. There is no build step and nothing to install.
