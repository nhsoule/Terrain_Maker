# Terrain Maker

Paint the pofa world map in a phone or desktop browser, preview it in 3D, and export it as a C header.

**Live:** https://nhsoule.github.io/Terrain_Maker/

The map is one grid with two layers:

- **Height**: grey shade = height. Tools: raise, lower, smooth, flatten.
- **Biome**: one flat color per pixel (water, sand, grass, forest, dirt, rock, snow). Tools: paint, fill, pick.

The 3D view on the right (or the 3D tab on a phone) is the actual flat-shaded mesh, Z-up, with the sea plane.

## Use

- One finger or left mouse paints. Two fingers or right-drag pans; pinch or the wheel zooms. In 3D: drag to orbit, pinch or wheel to zoom.
- **Generate** makes a seeded island or continent. **Auto biome** colors the map from height and slope (beach, rock, snow, forest).
- **Map** sets the grid size, meters per pixel, max height, sea level and chunk size, and shows chunk and mesh stats.
- **File** saves and loads a `.json` project, saves or imports height and biome PNGs, and starts a new map. Work also autosaves in the browser.
- **Export** writes `world.h` for pofa.

Keys: Ctrl+Z / Ctrl+Y undo and redo, `[` `]` change brush size, 1-7 pick a biome.

## Export format (`<name>.h`)

- **Constants:** `name_w`, `name_h`, `name_meters_per_px`, `name_max_height`, `name_height_scale`, `name_sea_level`, `name_chunk_cells`, `name_chunks_x`, `name_chunks_y`.
- **`enum TerrainBiome`** plus `terrain_biome_rgb[BIOME_COUNT][3]`.
- **`name_height[w*h]`:** `uint16_t`, or `uint8_t` if you pick 8-bit. Multiply by `name_height_scale` to get meters.
- **`name_biome[w*h]`:** `uint8_t`.

Data is row-major and row 0 is north. Each pixel is a vertex and each cell makes two CCW triangles split on the SW-NE diagonal. A cell's biome comes from its NW pixel. Chunks share edge samples, so they don't crack. The comments in the header spell out the exact layout.

Everything is one `index.html`. There is no build step and nothing to install.
