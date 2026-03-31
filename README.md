# Luminous Erosion — UE5 Terrain Generator

A browser-based procedural 3D terrain generator with hydraulic erosion simulation and full PBR material export pipeline for Unreal Engine 5.

**Made by Adam Hawil, as a personal research project.**

![UE5 Ready](https://img.shields.io/badge/UE5-Ready-blue?style=flat-square) ![PBR](https://img.shields.io/badge/PBR-Full%20Pipeline-cyan?style=flat-square) ![License](https://img.shields.io/badge/license-MIT-green?style=flat-square)

---

## Features

- **Procedural terrain** built from layered Perlin noise with fBm octaves and ridged noise blending
- **Hydraulic erosion** simulation — particle-based water droplets carve realistic channels and deposit sediment
- **Real-time 3D preview** using Three.js with orbit controls, shadows, and fog
- **Full PBR material export** — 6 texture maps ready for Unreal Engine 5
- **True 16-bit grayscale heightmap** — raw PNG with 65,536 height levels, no 8-bit banding
- **UE5-compatible resolutions** — export sizes match Unreal's landscape component grid
- **Randomize everything** — one click generates a completely unique terrain with randomized parameters, colors, and lighting
- **Wireframe mode** for inspecting mesh topology
- **Sun position controls** — adjustable azimuth, elevation, intensity, and color
- **Zero dependencies** — single self-contained HTML file, runs in any modern browser

---

## Quick Start

1. Download `luminous-erosion.html`
2. Open it in any modern browser (Chrome, Firefox, Edge, Safari)
3. That's it — no server, no install, no build step

Or clone the repo:

```bash
git clone https://github.com/YOUR_USERNAME/luminous-erosion.git
cd luminous-erosion
open luminous-erosion.html
```

---

## Usage

### Generating Terrain

- Adjust **Seed** to get reproducible terrain — same seed always produces the same landscape
- Use **Random Seed** to randomize all parameters at once (terrain, PBR, colors, lighting)
- Use **Prev/Next** to iterate through seeds while keeping your current slider settings
- Click **Regenerate** after changing sliders to rebuild the terrain

### Terrain Controls

| Control | Range | Description |
|---------|-------|-------------|
| Preview Resolution | 64–256 | Vertex density of the 3D preview mesh |
| Height Scale | 0.5–8.0 | Vertical exaggeration of the terrain |
| Noise Scale | 1.0–10.0 | Frequency of terrain features (higher = more peaks) |
| Octaves | 1–10 | Layers of noise detail (more = finer features) |
| Ridge Blend | 0–1.0 | Mix between smooth hills and sharp ridges |
| Erosion Drops | 0–15,000 | Number of water droplet simulations |
| Erosion Strength | 0.01–0.4 | How aggressively water carves the terrain |
| Water Level | 0–1.0 | Height of the transparent water plane |

### PBR Material Controls

| Control | Description |
|---------|-------------|
| Base Roughness | Default roughness for flat terrain |
| Erosion Roughness | Roughness in eroded channels (polished rock) |
| AO Intensity | Strength of the ambient occlusion in crevices |
| Color Ramp | 4-stop gradient: Valley → Lowland → Highland → Peak |

### Lighting

| Control | Description |
|---------|-------------|
| Sun Azimuth | Horizontal rotation of the sun (0°–360°) |
| Sun Elevation | Vertical angle — low = dramatic shadows, high = flat noon light |
| Sun Intensity | Brightness multiplier |
| Sun Color | Color temperature of the directional light |
| Ambient Color | Fill light color for shadowed areas |

### 3D Viewport

- **Left-click drag** — orbit the camera
- **Scroll wheel** — zoom in/out
- **Touch drag** — orbit on mobile
- **Pinch** — zoom on mobile
- **Wireframe toggle** — inspect mesh topology

---

## Exporting for Unreal Engine 5

### Export Resolutions

These match UE5's landscape component grid sizes:

| Resolution | UE5 Layout | Use Case |
|------------|-----------|----------|
| 256 × 256 | 4×4 Components (Small) | Prototyping, quick iteration |
| 505 × 505 | 1 Component 8×8 | Small terrain |
| 1009 × 1009 | 2×2 Components | Medium terrain (default) |
| 2017 × 2017 | 4×4 Components | Large terrain |
| 4033 × 4033 | 8×8 Components | Very large terrain |

### PBR Maps

Click individual map buttons or **Export All** to download all 6 maps at once:

| Map | Format | UE5 Material Input |
|-----|--------|--------------------|
| **Heightmap** | 16-bit Grayscale PNG | Landscape Import → Heightmap |
| **Normal Map** | 8-bit RGB (DirectX/DX convention) | Material → Normal |
| **Roughness** | 8-bit Grayscale | Material → Roughness |
| **Base Color** | 8-bit sRGB | Material → Base Color |
| **Ambient Occlusion** | 8-bit Grayscale | Material → Ambient Occlusion |
| **Erosion Mask** | 8-bit Grayscale | Landscape Layer Blend / Foliage Mask |

### Importing the Heightmap into UE5

1. Open your UE5 project
2. Go to **Landscape Mode** (Shift+3)
3. Select **Import from File**
4. Browse to `heightmap-16bit-seed*.png`
5. UE5 will auto-detect the 16-bit grayscale format
6. Set the resolution to match your export (e.g., 1009×1009)
7. Adjust the Z scale to taste

### Setting Up the PBR Material in UE5

1. Create a new **Material** in the Content Browser
2. Import the exported texture maps
3. Connect them:
   - `basecolor-seed*.png` → **Base Color** input
   - `normal-seed*.png` → **Normal** input (set as Normal Map in texture settings)
   - `roughness-seed*.png` → **Roughness** input
   - `ao-seed*.png` → **Ambient Occlusion** input
4. The **Erosion Mask** can be used as a landscape layer blend mask or to drive foliage density in eroded channels

### Normal Map Convention

The normal map uses **DirectX convention** (green channel inverted), which is what UE5 expects natively. No need to flip any channels.

---

## How It Works

### Terrain Generation

The terrain is built from **fractal Brownian motion (fBm)** — multiple octaves of Perlin noise layered at different frequencies and amplitudes. A second noise pass generates **ridged noise** (the absolute value of noise inverted and squared), which is blended in via the Ridge Blend parameter to create sharp mountain peaks.

### Hydraulic Erosion

The erosion algorithm simulates thousands of water droplets:

1. Each droplet spawns at a random position
2. It follows the steepest downhill gradient
3. Moving downhill, it erodes terrain and picks up sediment based on its speed and capacity
4. Moving uphill or slowing down, it deposits sediment
5. Water evaporates over time, reducing carrying capacity
6. The process repeats for up to 80 steps per droplet

This creates realistic drainage patterns, river valleys, and sediment deposits.

### 16-Bit Heightmap Encoder

Since the HTML Canvas API only supports 8-bit per channel, the heightmap uses a **custom raw PNG encoder** that constructs the binary PNG format directly — writing proper IHDR chunks with bit depth 16, color type 0 (grayscale), zlib-wrapped scanline data, and CRC32/Adler-32 checksums. This gives 65,536 height levels instead of 256, eliminating staircase banding on gentle slopes.

### PBR Map Generation

- **Normal map**: Computed from the height gradient using finite differences, output in tangent-space DirectX convention
- **Roughness**: Driven by slope steepness, erosion amount, and altitude (snow = smoother, cliffs = rougher, eroded channels = polished)
- **Base color**: 4-stop color ramp with slope-based rock blending and erosion channel darkening
- **Ambient occlusion**: Laplacian-based cavity detection that darkens crevices
- **Erosion mask**: Accumulated erosion values with square-root normalization for better visual distribution

---

## Browser Support

Tested on:
- Chrome 90+
- Firefox 90+
- Edge 90+
- Safari 15+

Requires WebGL support for the 3D viewport.

---

## Tech Stack

- **Three.js r128** — 3D rendering
- **Custom Perlin noise** — seeded, deterministic
- **Custom PNG encoder** — 16-bit grayscale binary construction
- **No build tools** — pure vanilla JS in a single HTML file

---

## License

MIT — use it however you want, appreciate all the feedback, tips to improve, ideas and credits!
