# VoxelVerse - a playable Minecraft in one HTML file

A browser voxel sandbox: infinite procedural terrain with caves and ores, block
breaking/placing, water, trees, a working light engine, pigs, TNT, rain, a synthesized
soundtrack of effects, an 8-minute day/night cycle and custom GLSL shaders (sky dome with sun
disc, moon and stars, procedural clouds, animated fresnel water, atmospheric fog, baked
ambient occlusion and per-face lighting, a first-person held block, and break particles).

Every world is defined by a single number. The same seed always rebuilds the same terrain, so
you can share a seed and meet in the same landscape.

The entire game is **`index.html`** - one file, no build step. three.js is loaded from a CDN;
everything else (engine, worldgen, mesher, physics, shaders, audio, touch UI) is inline.

## Play

| Host | Link |
| --- | --- |
| GitHub Pages | https://aniruddhaadak80.github.io/minecraft-web/ |
| Vercel | https://minecraft-web-aniruddha-adaks-projects.vercel.app |
| Vercel alias | https://minecraft-web-gules.vercel.app |

Add `#play` to skip the intro screen, e.g. `.../index.html#play`. Add `?seed=1337` to load a
specific world, e.g. `.../index.html?seed=1337#play`.

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
| Sound on/off | `M` |
| Toggle rain | `N` |
| Auto-step (climb one block without jumping) | `O` |
| Shoo the nearest pig | `K` |
| Pause / world menu (seed) | `Esc`, or click the clock |

The block you hold is shown in the corner and swings when you break or place. Breaking a
block or hitting a pig plays a matching synthesized sound; wind and rain beds fade in and out
with the weather.

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
| Rain on/off | Rain button |
| Select / assign blocks | Tap the hotbar, or the palette button for all blocks |
| Pause / world menu | Tap the clock in the top-right |

## Blocks

22 blocks: grass, dirt, stone, sand, **sandstone** (the layer under beaches), cobblestone,
gravel, mossy stone, planks, brick, **glass** (transparent), snow, wood, oak leaves, pine
needles, **coal ore** and **iron ore** (pockets hidden in the stone), **TNT**, plus poppies,
daisies and grass tufts (cross-shaped plants with wind sway), and a **lamp** that glows,
pulses, and lights up its surroundings.

World features: oceans and lakes with sand beaches, oak and pine trees, snow-capped
mountains, plants scattered on grass, water that settles into holes dug below sea level,
and **caves** - twin 3D-noise fields carve explorable tunnels under the surface, pitch dark
until you bring a lamp.

## Mobs, TNT and weather

- **Pigs** wander the grass, hop, flee when you get close, and respawn elsewhere a few seconds
  after you defeat them. They have three hit points, flash white when hit, and react to being
  shoved. A small ambient rig keeps them visible at night without a lamp.
- **TNT** is a placeable block: hit it and it hisses, blinks faster, then explodes. The blast
  carves a ragged sphere out of the terrain (smaller underwater), shakes the camera, throws you
  back, and lights the fuse of any TNT caught in the radius, so chains are easy to build.
- **Rain** rolls in on its own and can be toggled on `N`. 650 GPU points fall around you and
  recycle at the top; the sky greys out, clouds thicken, and a rain bed fades in under the
  wind. Water entry splashes, footsteps change with the surface, and an **auto-step** option
  lifts you over single blocks when you walk into them (on by default for touch).
- **Seed** - the terrain, caves, ores, trees and plants all derive from one integer seed. The
  pause overlay shows it, lets you type a new one, and offers a **New world** button. Edits are
  stored per seed, so starting a new world does not inherit the old one's changes.

## How it works

- **Worldgen** - value-noise fBm builds a heightmap (continents + hills + ridges); columns are
  filled with stone/dirt/grass/sand/sandstone, sea level carves oceans, snow caps grow above
  y=46. Two offset 3D value-noise fields carve caves below the surface, and a 3D hash scatters
  coal and iron ore pockets through the stone. Chunks are `16 x 16 x 80`. The seed is XORed into
  every hash, so one integer reproduces the entire world.
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
  fly mode. Plants never collide; glass does; sneaking blocks you from stepping off a ledge;
  auto-step lifts you one block when a step stops horizontal motion.
- **Mobs** - box-vs-voxel collision shared with the player, a tiny wander/flee/hop state
  machine, gravity and snapping to the surface, and a generous hit cone from the crosshair
  that is blocked by the world.
- **Explosions** - a radius scan with a noise-perturbed radius removes blocks, drops the light
  map and rebuilt chunk meshes behind it, applies radial knockback to the player and nearby
  pigs, and queues any TNT inside the radius.
- **Weather and audio** - rain is a single `Points` cloud recycled around the player with an
  intensity that eases in and out; sounds are generated with an `OscillatorNode`/noise-buffer
  graph per event, so there are no audio files, and everything stays silent until the first
  user gesture unlocks the context.
- **Presentation** - first-person held-block view model with walk bob and a swing on use, GPU
  break particles, an underwater tint, sprint/zoom FOV easing, an explosion shake, and a custom
  crosshair cursor.
- **Persistence** - blocks you change are written to `localStorage` under a key derived from
  the seed (60k edits max), so reopening the page restores your build and a different seed
  starts clean.
- **Streaming** - chunks generate/build around the player and unload beyond render distance;
  edits mark the touched chunk plus its neighbours dirty for a rebuild.

## Run locally

Just open `index.html` in a browser, or serve the folder:

```bash
npx serve .          # or: python -m http.server
```
