# VoxelVerse — a playable Minecraft in one HTML file

A browser voxel sandbox: infinite procedural terrain, block breaking/placing, water, trees,
an 8-minute day/night cycle and custom GLSL shaders (sky dome with sun disc + stars, procedural
clouds, animated fresnel water, atmospheric fog, baked ambient occlusion).

The entire game is **`index.html`** — one file, no build step. three.js is loaded from a CDN;
everything else (engine, worldgen, mesher, physics, shaders) is inline.

## Play

| Host | Link |
| --- | --- |
| GitHub Pages | https://aniruddhaadak80.github.io/minecraft-web/ |
| Vercel | https://minecraft-web-<hash>.vercel.app |

Add `#play` to skip the intro screen, e.g. `.../index.html#play`.

## Controls

| Action | Keys |
| --- | --- |
| Move | `W` `A` `S` `D` |
| Jump / swim up | `Space` |
| Sprint | `Shift` |
| Look | Mouse (click the canvas to capture the pointer) |
| Break block | Left click (hold to repeat) |
| Place block | Right click (hold to repeat) |
| Select block | `1`–`6` or mouse wheel |
| Fly mode | `F` (then `Space` up / `Shift` down) |

## How it works

- **Worldgen** — value-noise fBm builds a heightmap; columns are filled with stone/dirt/grass/sand,
  sea level carves oceans and beaches, and trees are scattered on grass. Chunks are `16 × 16 × 80`.
- **Mesher** — per-chunk greedy-free face extraction: only faces touching air/water are emitted, with
  per-vertex ambient occlusion (0–3 occluders) and per-face directional shading baked into vertex colours.
- **Shaders** — terrain, water, sky and clouds each get a `ShaderMaterial`:
  directional sun + hemisphere ambient + distance fog + hash-based surface noise; water adds vertex-wave
  displacement, fresnel and a Blinn sun glint; the sky dome has a gradient, sun disc, halo and twinkling
  stars; clouds are a distance-faded fBm layer.
- **Physics** — sweep-and-prune style AABB vs voxel collision, axis-separated resolution, gravity,
  swimming and fly mode.
- **Streaming** — chunks generate/build around the player and unload beyond render distance;
  edits mark the touched chunk plus its neighbours dirty for a rebuild.

## Run locally

Just open `index.html` in a browser, or serve the folder:

```bash
npx serve .          # or: python -m http.server
```
