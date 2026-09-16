<div align="center">

# Fractalis.

**Observatory of Infinite Objects: real-time GLSL raymarching of 3D fractals in a single HTML file**

<img width="2154" height="1131" alt="image" src="https://github.com/user-attachments/assets/d63ce682-ebfb-4b4a-b53b-0f8a3af1431b" />

<img width="2154" height="1132" alt="image" src="https://github.com/user-attachments/assets/4535cc1e-a4f8-42e5-8075-bda2c28887bc" />

<img width="2153" height="1123" alt="image" src="https://github.com/user-attachments/assets/91be3f25-9326-4ea1-acf6-1bc652c776bc" />

</div>

---

## Overview

**Fractalis** is a self-contained, browser-based explorer for 3D fractals. Every specimen is rendered on the GPU by a fragment shader that raymarches a distance estimator in real time, with soft shadows, ambient occlusion, orbit-trap coloring, relative fog and ACES tone mapping.

No install, no build step, no bundler: open the HTML file and start exploring. The whole application (UI, shaders, camera system, renderer) lives in one file, with Three.js loaded from a CDN.

## Features

- **8 fractal specimens**, each with its own tuned distance estimator and parameters
- **Live parameter editing** with sliders (power, scale, folds, rotations, quaternion constants, iterations)
- **Orbit camera** with smoothed inertia, plus a **free-flight camera** (WASD / ZQSD) to dive inside the structures
- **5 cosine palettes** with cycle speed, orbit-trap weight and phase controls
- **Lighting controls**: light azimuth and elevation, soft shadows, ambient occlusion, fog density
- **3 quality presets** (Draft, Balanced, Fine) with **automatic quality downgrade** when the frame rate drops
- **Parameter drift**: slow, animated evolution of one key parameter per fractal
- **Random specimen** generator using curated "safe" ranges so results stay attractive
- **High-resolution PNG capture** (rendered at 2x the current pixel ratio, capped around 36 MP)
- **Cinema mode** hides the entire interface for clean viewing or screen recording
- **Touch support**: one-finger orbit, two-finger pinch zoom
- **Live stats**: FPS, frame time, render resolution, raymarch step count
- **Robustness**: WebGL context loss / restore handling, guarded boot with readable error messages

## Specimens

| No. | Name | Description | Tunable parameters |
|:---:|------|-------------|--------------------|
| 01 | **Mandelbulb** | The mathematical orchid | Power, Flux (animated domain warp), Iterations |
| 02 | **Quaternion Julia** | An infinity pinned by c | C real part, C i axis, C j axis, C k axis, Iterations |
| 03 | **Mandelbox** | The folding factory | Scale, Fold, Twist xy, Twist yz, Iterations |
| 04 | **Menger Sponge** | The cube with infinite windows | Scale, Roll, Pitch, Yaw, Iterations |
| 05 | **Sierpinski Tetrahedron** | The triangle raised to volume | Scale, Rotation alpha / beta / gamma, Iterations |
| 06 | **Apollonian Spheres** | The tangent packing | Density, Tilt, Shear, Iterations |
| 07 | **Box x Bulb Hybrid** | A cross of two infinities | Power, Scale, Cubic phase, Iterations |
| 08 | **Sierpinski Universe** | The tetrahedral kaleidoscope | x / y / z axis offsets, Spiral, Iterations |

## Getting started

Then simply open `Fractalis_01.html` in a modern browser (Chrome, Edge, Firefox, Safari).

An internet connection is required on first load to fetch Three.js from cdnjs. 
For fully offline use, download `three.min.js` (r128) next to the HTML file and update the `<script>` tag accordingly.

> A dedicated GPU is recommended. Fractal raymarching is fragment-heavy: every pixel runs up to several hundred distance evaluations per frame.

## Controls

### Orbit mode (default)

| Input | Action |
|-------|--------|
| Left drag | Orbit around the object |
| Mouse wheel | Zoom |
| Two-finger pinch | Zoom (touch) |
| `Space` | Pause / resume auto rotation |
| `R` | Reframe (reset camera, exits free flight) |

### Free camera mode

| Input | Action |
|-------|--------|
| `V` | Toggle free camera |
| `W` `A` `S` `D` (or `Z` `Q` `S` `D`, arrows) | Move |
| Hold mouse + drag | Look around |
| `Space` / `E` | Move up |
| `C` | Move down |
| `Shift` | Speed boost (x4) |
| Mouse wheel | Adjust flight speed |

### Interface

| Key | Action |
|-----|--------|
| `H` | Toggle cinema mode (hide all UI) |
| `P` | Fold / unfold the side panel |

Shortcuts are ignored while `Ctrl`, `Cmd` or `Alt` is held, and `Space` / arrow keys are not captured while a slider or button has focus.

## Side panel

| Section | Content |
|---------|---------|
| **I. Specimens** | Fractal selection with animated cross-fade |
| **II. Geometry** | Per-fractal parameters |
| **III. Color** | Palette (Ember, Glacier, Ivory, Forest, Nebula), Cycle, Orbit trap, Phase |
| **IV. Light** | Azimuth, Elevation, Shadows, Occlusion, Fog |
| **V. Engine** | Quality preset, Auto rotation, Parameter drift |

Action buttons: **Capture PNG**, **Free camera**, **Random specimen**, **Cinema mode**.

## Quality presets

| Preset | Max raymarch steps | Pixel ratio |
|--------|:------------------:|:-----------:|
| Draft | 110 | 1.0 |
| Balanced (default) | 190 | 1.5 |
| Fine | 300 | 2.0 |

If the frame rate stays under 13 FPS for about 2.5 seconds (after a short warm-up), Fractalis automatically steps down one preset and notifies you.

## Rendering pipeline

Fractalis draws a single full-screen quad with an orthographic camera. All the work happens in the fragment shader, which is assembled per specimen:

```
FRAG_HEAD  (uniforms, helpers: rot2, quaternion multiply)
   +
DE_CHUNK   (one deFractal() distance estimator per fractal)
   +
FRAG_TAIL  (normals, shadows, AO, march, palette, shading, post)
```

Compiled shader sources are cached, so switching specimens only builds each variant once.

Key techniques:

- **Sphere tracing** with a distance-relative hit epsilon and a `-1` miss sentinel
- **Bounding sphere culling**: rays are intersected with a per-fractal bounding sphere, and marching only runs between entry and exit points (disabled for space-filling fractals such as Apollonian and Universe)
- **Early escape** in the Mandelbox once the orbit diverges, for a 3 to 10x speed-up
- **Tetrahedral normal estimation** (4 samples instead of 6)
- **Soft shadows** via penumbra estimation and **5-tap ambient occlusion**
- **Orbit-trap coloring** mapped through Inigo Quilez style cosine palettes
- **Multi-term lighting**: key light, sky fill, ground bounce, rim light and specular
- **Relative fog** measured beyond 60% of the framing distance, so the front of the object always stays crisp
- **ACES filmic tone mapping**, gamma correction, animated film grain
- **Procedural twinkling starfield** background
- GLSL loop bounds are set slightly above each iteration slider maximum, so every setting is truly reachable under WebGL 1 constant-loop rules

## Project structure

```
fractalis/
├── Fractalis_01.html   # The complete application
├── docs/
│   └── screenshot.png  # Optional README image
└── README.md
```

## Browser requirements

- WebGL 1.0 support with hardware acceleration enabled
- ES6 JavaScript (template literals, arrow functions, `Set` / `Map`)
- Pointer Events API for mouse and touch input

If WebGL or Three.js cannot be loaded, Fractalis displays an explicit error screen instead of a blank page.

## Changelog

### v6.1
- **Critical fix**: a GLSL comment inside a template literal contained backticks, which broke script parsing (black screen). All template literals were audited.

### v6
- Iteration caps raised (Mandelbulb 32, Julia 24, Mandelbox 24, Menger 20, Sierpinski 24, Apollonian 16, Hybrid 24, Universe 24) with matching GLSL loop bounds
- Hit test now uses a `-1` sentinel, so hits at `t = 0` no longer fall back to the background
- Slider readouts derive their decimal count from the slider step
- The randomizer no longer changes the iteration count
- Keyboard shortcuts ignored under modifier keys; `Space` and arrows no longer hijacked on focused controls

### v5
- Mandelbox bounding sphere test and early escape
- Fog made relative to the framing distance

## Credits

- Built with [Three.js](https://threejs.org/) (r128)
- Typography: [Fraunces](https://fonts.google.com/specimen/Fraunces) and [IBM Plex Mono](https://fonts.google.com/specimen/IBM+Plex+Mono)
- Distance estimation and palette techniques inspired by the work of Inigo Quilez, Mikael Hvidtfeldt Christensen (Syntopia) and the Fractal Forums community

Created by **Creepy Cat** / [Creepy Cat](https://creepycat.fr)

## License: MIT
