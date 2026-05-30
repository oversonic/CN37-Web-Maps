# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Interactive map viewer for the CN37 housing project (โครงการ CN37) — a Thai residential development with 495 units. The app overlays a vector floor plan on a live geographic base map (OpenStreetMap/Esri satellite) with pan/zoom/rotate navigation and 32 CCTV camera markers.

## Files

| File | Purpose |
| --- | --- |
| `CN37_masterplan_inline.html` | **Primary app** — fully self-contained, no build step, no external files |
| `CN37_masterplan.html` | Legacy version (loads SVG via fetch, kept for reference) |
| `CN37_MasterPlan3_cctv.svg` | Source SVG from Adobe Illustrator (not loaded at runtime in inline version) |

## Running

`CN37_masterplan_inline.html` opens directly as `file://` — no server needed (SVG is inline).

For the legacy `CN37_masterplan.html`, a static server is required:

```
python -m http.server
```

## Architecture

### Layer stack (z-index order)

```
z-index 3  #cam-html-layer   — 32 camera markers (HTML divs, always on top)
z-index 2  #svg-overlay      — SVG floor plan (pointer-events:none)
z-index 1  #lmap             — Leaflet base map (OSM / Esri satellite)
```

### Base map — Leaflet.js

- `leaflet@1.9.4` + `leaflet-rotate@0.2.8` loaded from CDN
- 3 tile layers switchable via bottom-right control: OSM, OSM HOT, Esri satellite
- Map rotation via `map.setBearing(deg)` from `leaflet-rotate`
- Zoom/fit buttons wire directly to `map.zoomIn()` / `map.fitBounds(PROJECT_BOUNDS)`

### SVG floor plan overlay

The inline SVG (`id="plan"`, viewBox `0 0 4757.904 1901`) is positioned using a **2-point similarity transform** (scale + rotation, no shear):

```
Tie point 1: SVG (201, 851)  ↔  GPS (13.9180573, 100.5866030)  — ป้อม รปภ. หน้า
Tie point 2: SVG (4558, 897) ↔  GPS (13.9143259, 100.5952891)  — ป้อม รปภ. หลัง
```

`syncOverlay()` runs on every `map.on('move zoom viewreset')` and after every `setBearing()` call. **leaflet-rotate does NOT fire a Leaflet event on bearing change** — `syncOverlay()` must be called manually from `setBearing()`. It computes `matrix(a, b, -b, a, tx, ty)` applied to the SVG element's `style.transform`.

### CCTV camera markers

- 32 cameras (CAM-01–CAM-32), coordinates from `KML_TheConnect_cctv` (exact GPS)
- Rendered as `<div class="lmk-wrap">` elements inside `#cam-html-layer` (z-index:3, above floor plan)
- Positions updated via `map.latLngToContainerPoint(ll)` on every map move/zoom/rotate
- `window._updateCamPos()` is called from `setBearing()` to sync after rotation
- Toggle visibility via sidebar "กล้องวงจรปิด" panel

### Sidebar controls

| Section | What it does |
| --- | --- |
| ประเภทที่อยู่อาศัย | Legend (visual only — SVG has no `data-key` attributes) |
| การแสดงผล | Toggle base map visibility / toggle SVG white background (`#BG` group) |
| กล้องวงจรปิด | Show/hide all 32 camera markers |

### Viewer controls (top-right bar)

- **ผังโครงการ** slider — opacity of `#svg-overlay` (0–100%)
- **หมุน** ↺ / ↻ / ⊙ — rotate map ±15° / reset bearing

## Design Tokens

All colors are CSS custom properties on `:root`:

- `--c-th1` (#3a93f0) — Townhome 1 parking
- `--c-th2` (#21c0b6) — Townhome 2 parking
- `--c-d-sm` (#7c8a82) — Small duplex
- `--c-d-lg` (#f0922e) — Large duplex
- `--c-office` (#e8388f) — Juristic office
- `--c-guard` (#8d1f3a) — Guard post

## Key Numbers (hardcoded in HTML)

- **495 units** total (header stat)
- **28 sois**
- **2 guard posts**
- **32 CCTV cameras**
- Project bounds: `[[13.9130, 100.5850], [13.9195, 100.5975]]`

## SVG Layer Structure

Hidden layers were stripped when building the inline version. Active layers in the inline SVG:

| Layer id | Content |
| --- | --- |
| `BG` | White background rectangle — toggled by sidebar button |
| `cover` | Project boundary polylines |
| `line` | House floor plan shapes (white/gray fills) |
| `compass` | Compass rose |
| `number` | House unit number circles |
| `soiNum` | Soi number labels |
| `guard-booths` | Two dark-red guard post rectangles (actual map positions) |

Removed at build time: `spece`, `line2Outline`, `EleT`, `ELeLine`, `EleTxt` (all were `display="none"`), and `discription` (legend box — replaced by sidebar).
