# 〰️ Seismic Acquisition Simulator

A browser-based interactive tool for exploring marine seismic reflection surveys — from placing shots and receivers to seeing synthetic seismograms, CMP fold maps, and azimuth roses update in real time.

**No installation. No backend. No dependencies.** Open the HTML file directly in any modern browser — it works completely offline.

---

👉 **[Link to the app](https://frasera00.github.io/SeismicApp/)**

---

## What It Does

Place shots and receivers in any 2D or 3D geometry and immediately see:

- Synthetic seismograms computed in real time using **Snell's law** and the **image-source method**
- **Water-bottom reflections**, deep reflector arrivals, and **free-surface ghosts**
- **NMO correction** with RMS velocity computed from the layer model
- **AVO** (Amplitude vs. Offset) with configurable R₀ and gradient G
- **CMP fold maps** and **azimuth & offset rose plots**
- Shot, receiver, and **CMP gather** sorting (including a true CMP stack mode)
- Gather filters, axis controls, and a zoom-in modal for detailed inspection

---

## Acquisition Geometries

The default configuration loads a **2D OBN setup** — one OBN at the water bottom, 1km down, and 20 shots spaced 200 m inline. You can also build:

**Streamer**
  *Single cable* — one inline array of receivers towed behind one shot  
          
          Z ↑
            └─ → X   ◆shot ──●rec ──●rec ──●rec ──●rec

          Y ↑
            └─ → X   ◆shot ──●rec ──●rec ──●rec ──●rec        
 
    Settings:
    - Shots: IL × XL = 1 × 1 · origin (0, 0, 0)
    - Receivers: IL × XL = N × 1 · origin (Δx, 0, 0) · IL spacing = Δx

  *Multi-cable* — one shot, multiple parallel receiver cables offset in Y:

          Z ↑
            └──→ X/Y ◆shot ──●rec ──●rec ──●rec ──●rec ← all cables at z = 0
            
          Y ↑               ┌──●rec ──●rec ──●rec ──●rec ← cable 1 (Y = +Δy)
            └─ → X   ◆shot |──●rec ──●rec ──●rec ──●rec ← cable 2 (Y = 0)
                            └──●rec ──●rec ──●rec ──●rec ← cable 3 (Y = −Δy)

    Settings:
    - Shots: IL × XL = 1 × 1 · origin (0, 0, 0)
    - Receivers: IL × XL = N × M · origin (Δx, −(M−1)·Δy/2, 0) · IL spacing = Δx · XL spacing = Δy
                                 
-**OBN (Ocean Bottom Node)**
Receivers at z = water bottom depth, shot grid near surface above |
  Full 3D illumination — every azimuth is recorded at each node.

          Z ↑
            └─ → X      ◆shot ──◆shot ──◆shot ──◆shot       ↑     near surface (z ~ 0)
                                                              |
                                                              |
                          ──●rec ──●rec ──●rec ──●rec         ↓     water bottom (z = WB)


          Y ↑
            └─ → X      ◆shot ──◆shot ──◆shot ──◆shot      --> same for receivers but with 
                        ◆shot ──◆shot ──◆shot ──◆shot          origin at water bottom 
                        ◆shot ──◆shot ──◆shot ──◆shot

    Settings:
    - *Shot*: IL × XL = N × M · origin (0, 0, 0) · IL spacing = Δx · XL spacing = Δy · depth = 0
    - *Receivers*: IL × XL = P × Q · origin (ox, oy, WB) · IL spacing = Δrx · XL spacing = Δry · **depth z = water bottom value**
---

## Features

### Geometry Editor
- Depth (cross-section) and XY (top-down map) views
- Drag-and-drop shots and receivers; box selection; multi-select with Ctrl/Cmd
- Generate regular shot and receiver arrays (inline × crossline grids)
- Drag water-bottom and reflector interface lines directly on the canvas
- Import / export geometry as CSV

### Physics
- Water-bottom reflection via image-source method (exact)
- Deep reflector travel times via **N-layer Snell's law** (bisection solver)
- Direct arrival, all primary reflections, and free-surface ghosts that can be turned on/off
- AVO amplitude scaling per arrival using Shuey's two-term approximation

### Seismic Plots
- Three panels: **Initial**, **Current**, and **Difference** (change since initial state)
- X-axis options: Offset, Shot X/Y, Receiver X/Y, **CMP X/Y**
- CMP gather mode bins and stacks traces by midpoint using the CMP bin size
- NMO correction toggle, gain control, and time range selector
- Click any panel to open a zoom modal with drag-to-zoom and right-click zoom-out

### Analysis
- **Azimuth & offset rose** — polar histogram of all shot-receiver azimuths weighted by offset
- **CMP fold map** — 2D grid showing trace count per bin
- Selection-aware: selecting objects in the editor instantly updates all four plots and the analysis panels

### Tutorial
A built-in 20-step interactive tutorial covers everything from placing your first shot to understanding CMP stacking, NMO, AVO, and the difference between streamer and OBN acquisition — with SVG diagrams.

---

## Getting Started

1.
```bash
# Clone the repository
git clone https://github.com/your-username/seismic-app.git

# Open the app — no build step needed
open seismic-app.html
```
2.
Download `seismic-app.html` and open it in Chrome, Firefox, or Safari.

3.
**[Access it online](https://frasera00.github.io/SeismicApp/)**
---

## How It's Built

The entire application is a **single self-contained HTML file** (~186 KB) with no external dependencies at runtime:

| Layer | Technology |
|---|---|
| Structure | HTML5 |
| Styling | Vanilla CSS (custom properties, dark/light theme) |
| Logic & rendering | Vanilla JavaScript (ES2020+) |
| Drawing | HTML5 Canvas 2D API |

No frameworks, no bundler, no npm. The physics engine, ray tracer, synthetic seismogram renderer, geometry editor, and all plots are written from scratch in plain JS.

### Architecture Overview

```
state {}                  ← single source of truth (geometry, model, UI state)
    │
    ├── renderEditor()    ← Canvas 2D: geometry view (XY or depth cross-section)
    ├── refreshPlots()    ← buildSynthetic() → drawPlot() × 3 panels
    ├── drawRosePlot()    ← azimuth × offset polar histogram
    └── drawCMPFold()     ← CMP bin fold map
```

`buildSynthetic()` loops over every shot–receiver pair, computes travel times for each arrival type, and sums Ricker wavelets into wiggle traces. Everything re-renders on every state change.

---

## Physics Details

- **Direct arrival**: straight-line distance / water velocity
- **Water-bottom reflection**: image-source method (exact for flat WB)
- **Reflector arrivals**: N-layer Snell's law solved with a 64-iteration bisection on the horizontal slowness *p*
- **Free-surface ghosts**: delay-based model — each primary arrival is duplicated with a polarity-reversed copy shifted by `2 × depth / Vwater`
- **NMO**: hyperbolic correction using RMS velocity computed from the layer stack
- **AVO**: `A(θ) = R₀ + G·sin²θ` (Shuey two-term), angle estimated from offset and total reflector depth

---

## Contributing

Issues and pull requests are welcome. The codebase is intentionally kept in a single file for portability — if you're extending it, please maintain that constraint.

---

## License

MIT — free to use, modify, and distribute.
