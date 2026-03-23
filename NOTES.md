# Planet Labs — Orbital Constellation Viewer

## What This Is
A single-file interactive 3D visualization of Planet Labs' satellite constellation, built with Three.js (r128). Shows all five satellite types orbiting a realistic Earth with real geographic data.

## Tech Stack
- **Three.js r128** via CDN (`cdnjs.cloudflare.com`)
- **TopoJSON Client v3** via CDN (`jsdelivr.net`) — for parsing land geometry
- **Natural Earth 50m land data** from `world-atlas@2` (falls back to 110m)
- Single HTML file, no build step, no dependencies to install

## Earth Rendering
- 2048×1024 canvas texture generated at runtime
- Ocean: dark blue gradient with subtle procedural noise
- Land: drawn from real TopoJSON polygons with latitude-based biome coloring (tropical → temperate → boreal → ice) and longitude checks for desert regions (Sahara, Arabia, Australia)
- Coastal shelf glow: multi-pass additive stroke around coastlines
- Custom GLSL shader for lighting: Fresnel rim atmosphere, ocean specular highlights, soft diffuse falloff
- Separate atmosphere shell mesh (slightly larger sphere with rim glow shader)

## Satellite Constellations

| Name       | Color    | Altitude | Count | Shape    | Notes |
|------------|----------|----------|-------|----------|-------|
| SuperDove  | `#22d3ee`| 500 km   | 200   | CubeSat  | Uses **InstancedMesh** for performance (200 sats) |
| SkySat     | `#f97316`| 450 km   | 21    | MiniSat  | Individual meshes, 3 orbital planes |
| Pelican    | `#a78bfa`| 475 km   | 6     | SmallSat | Next-gen, NVIDIA Jetson AI edge compute |
| Owl        | `#facc15`| 510 km   | 4     | Owl      | Custom model with large optic dome |
| Tanager    | `#34d399`| 522 km   | 2     | Tanager  | Hyperspectral, 420 bands, methane/CO₂ detection |

## 3D Satellite Models
Each type has a hand-built Three.js geometry model using composed primitives (boxes, spheres):
- `buildCubeSat` — 3U body + 4 solar panels + antenna + lens
- `buildSkySat` — Larger body + gold front face + 2 wing panels + antenna mast
- `buildSmallSat` — Bigger body + 4 solar wing panels + dish antenna + lens
- `buildOwl` — Wide body + 2 large panels + prominent optic sphere + radiator
- `buildTanager` — Body + spectrometer module + gold calibration panel + 4 solar wings + lens

Materials: `MeshPhongMaterial` throughout — dark blue solar panels (`pM`), gold foil (`gM`), black lens (`lM`), aluminum antenna (`aM`), tinted body color per constellation.

## Orbital Mechanics
- Sun-synchronous orbits, inclination ~97°
- Satellites distributed across `planes` orbital planes with phase offsets
- Orbit speed scales inversely with altitude (Kepler-ish)
- Orbital path rings drawn as `LineLoop` with low-opacity constellation color
- Each sat uses `lookAt` along its velocity vector for correct orientation

## UI Features
- **Toggle panel** (top right): click to show/hide each constellation
- **Speed slider** (bottom right): 0.1× to 5× time multiplier
- **Info panel** (bottom left): active sat count, orbit type, controls hint
- **Hover tooltips**: raycasting on satellites shows name, description, altitude, fleet size, resolution
- **Camera**: spherical coords, drag to rotate, scroll to zoom, slow auto-rotate when idle
- **Loading screen**: progress bar during Earth texture generation + TopoJSON fetch

## Stars
- 3000 points on a distant sphere shell (r = 80–200)
- Custom shader with size attenuation and sinusoidal twinkle animation

## Known Considerations
- All in one file (~336 lines) — could be split into modules
- Three.js r128 is older; some newer APIs (e.g., `CapsuleGeometry`) not available
- TopoJSON fetch requires network; has fallback chain (50m → 110m → none)
- SuperDove instanced mesh uses a simplified merged geometry (not the full CubeSat model)
