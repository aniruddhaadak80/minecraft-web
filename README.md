# VoxelVerse - a playable Minecraft in one HTML file

A browser voxel sandbox: infinite procedural terrain with caves and ores, block
breaking/placing, water, trees, a working light engine, an 8-minute day/night cycle and
custom GLSL shaders (sky dome with sun disc + stars, procedural clouds, animated fresnel
water, atmospheric fog, baked ambient occlusion and per-face lighting, a first-person held
block, and break particles).

The entire game is **`index.html`** - one file, no build step. three.js is loaded from a CDN;
everything else (engine, worldgen, mesher, physics, shaders, touch UI) is inline.

## Play

| Host | Link |
| --- | --- |
| GitHub Pages | https://aniruddhaadak80.github.io/minecraft-web/ |
| Vercel | https://minecraft-web-aniruddha-adaks-projects.vercel.app |

Add `#play` to skip the intro screen, e.g. `.../index.html#play`.

## Controls

### Desktop (mouse, trackpad, keyboard)
| Action | Keys |
| --- | --- |
| Move | `W` `A` `S` `D` or arrow keys |
| Look | Mouse - click the canvas to capture the pointer; dragging also works, with a custom crosshair cursor when unlocked |
| Jump / swim up | `Space` |
| Sprint | `Shift`, or double-tap `W` to lock sprint |
| Sneak (slower, camera lower, no walking off ledges) | `Ctrl` or `C` |
| Zoom | `Z` |
| Break block | Left click (hold to repeat) |
| Place block | Right click (hold to repeat) |
| Fly mode | `F` (`Space` up, `Shift` down) |
| Select block | `1`-`9` or mouse wheel |
| Block palette | `E` |
| Invert look | `I` |
| Look sensitivity | `[` / `]` |
| Time speed 1x/2x/4x/paused | `T` |
| Respawn | `R` |
| Release mouse | `Esc` |

The block you hold is shown in the corner and swings when you break or place.

### Phone / tablet
Touch controls appear automatically on touch devices (layout verified overlap-free from
360x640 up to tablet sizes):

| Action | Control |
| --- | --- |
| Move | Left virtual joystick (push to the rim to sprint) |
| Look | Drag anywhere on the right side |
| Jump | Jump button |
| Sneak | Sneak button |
| Zoom | Zoom button |
| Break block | Break button (hold to repeat) |
| Place block | Place button |
| Fly mode | Fly button |
| Select / assign blocks | Tap the hotbar, or the palette button for all blocks |

## Blocks

20 blocks: grass, dirt, stone, sand, cobblestone, gravel, mossy stone, planks, brick,
**glass** (transparent), snow, wood, oak leaves, pine needles, **coal ore** and **iron ore**
(pockets hidden in the stone), plus poppies, daisies and grass tufts (cross-shaped plants
with wind sway), and a **lamp** that glows, pulses, and lights up its surroundings.

World features: oceans and lakes with sand beaches, oak and pine trees, snow-capped
mountains, plants scattered on grass, water that settles into holes dug below sea level,
and **caves** - twin 3D-noise fields carve explorable tunnels under the surface, pitch dark
until you bring a lamp. Your edits are saved to `localStorage` and restored the next time
you load the page.

## How it works

- **Worldgen** - value-noise fBm builds a heightmap (continents + hills + ridges); columns are
  filled with stone/dirt/grass/sand, sea level carves oceans, snow caps grow above y=46. Two
  offset 3D value-noise fields carve caves below the surface, and a 3D hash scatters coal and
  iron ore pockets through the stone. Chunks are `16 x 16 x 80`.
- **Light engine** - every chunk carries a per-cell light map: skylight floods down each
  column (dimmed by leaves, water and glass), and lamps spread block light by BFS flood fill.
  Light is computed after trees so canopies shade correctly, and recomputed whenever a chunk
  is rebuilt - digging a skylight or placing/removing a lamp updates the lighting live.
- **Mesher** - per-chunk face extraction into four layers: opaque, water, glass and plants.
  Only faces touching air/water/plants are emitted, with per-vertex ambient occlusion (0-3
  occluders), per-face directional shading, and the baked light level multiplied into every
  vertex colour. Plants are two crossed quads displaced by a wind term in the vertex shader.
  Emissive blocks carry vertex colours above 1.0 and bypass lighting in the shader.
- **Shaders** - one `ShaderMaterial` per layer: terrain/glass share directional sun + hemisphere
  ambient + hash-based surface noise + distance fog; water adds vertex-wave displacement, fresnel
  and a Blinn sun glint; the sky dome has a gradient, sun disc, halo and twinkling stars; clouds
  are a distance-faded fBm layer that drifts and tints with the light.
- **Physics** - sweep AABB vs voxel collision, axis-separated resolution, gravity, swimming and
  fly mode. Plants never collide; glass does; sneaking blocks you from stepping off a ledge.
- **Presentation** - first-person held-block view model with walk bob and a swing on use, GPU
  break particles, an underwater tint, sprint/zoom FOV easing, and a custom crosshair cursor.
- **Streaming** - chunks generate/build around the player and unload beyond render distance;
  edits mark the touched chunk plus its neighbours dirty for a rebuild.

## Run locally

Just open `index.html` in a browser, or serve the folder:

```bash
npx serve .          # or: python -m http.server
```
