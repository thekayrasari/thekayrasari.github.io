---
title: "E6B Flight Computer"
date: 2026-03-06
tags: ["aviation", "flight planning", "navigation", "aeronautics"]
author: ["Kayra Sari"]
description: "A browser-based digital flight computer modeled after the classic E6B circular slide rule. Covers wind correction, time/speed/distance, fuel planning, true airspeed, density altitude, crosswind components, and top of descent — all computed client-side with a live vector diagram."
summary: "A full-featured digital E6B built in vanilla JavaScript, replicating both the wind side and calculator side of the classic aviation flight computer with real-time outputs and an interactive wind vector canvas."
---

> Live app: [thekayrasari.github.io/e6b](https://thekayrasari.github.io/e6b/) · [Source on GitHub](https://github.com/thekayrasari/e6b)

---

## Abstract

The E6B is the circular slide rule pilots have used for pre-flight planning since the 1940s. This project is a full-featured digital implementation built in vanilla JavaScript, replicating both sides of the physical device: the wind side for heading and groundspeed calculations, and the calculator side for time/speed/distance, fuel, airspeed, altitude, crosswind, and unit conversions.

The entire application is a single self-contained `index.html` file — no framework, no build step, no dependencies. A Canvas 2D wind vector diagram renders the triangle of velocities in real time as inputs change.

---

## How It Works

The app is split into two panels mirroring the physical E6B.

**Wind Side** computes the wind correction angle (WCA), true heading, and groundspeed using vector resolution. Inputs are true course, TAS, wind direction, and wind speed. A live canvas diagram draws the full velocity triangle — true course, TAS vector, wind vector, and groundspeed resultant — updating on every keystroke. Magnetic and compass headings are derived by applying variation and deviation on top of the true heading result.

**Calculator Side** is a tabbed panel with seven modules:

- **TSD** — solves for any one of time, speed, or distance given the other two
- **Fuel** — computes endurance, fuel required, and reserve from flow rate and onboard quantity
- **TAS** — converts CAS to true airspeed via density altitude, also outputs Mach number and ISA deviation
- **Altitude** — derives pressure altitude and density altitude from indicated altitude, QNH, and OAT
- **Crosswind** — resolves headwind and crosswind components for a given runway heading
- **TOD** — calculates top-of-descent distance for a standard 3° profile
- **Units** — converts between nm/sm/km, kg/lbs, litres/US gal/Imp gal, °C/°F, hPa/inHg, ft/m

---

## Key Equations

**Wind Correction Angle**

```
WCA = arcsin( Vw × sin(θ) / TAS )
GS  = TAS × cos(WCA) − Vw × cos(θ)
TH  = TC ± WCA
MH  = TH ∓ Variation
CH  = MH ∓ Deviation
```

where θ is the wind angle relative to the true course.

**True Airspeed**

```
ISA_dev  = OAT − (15 − 1.98 × PA/1000)
Dens_alt = PA + 120 × ISA_dev
ρ/ρ₀     = (1 − 6.8756×10⁻⁶ × DA)^4.2559
TAS      = CAS / √(ρ/ρ₀)
Mach     = TAS / (661.47 × √((OAT + 273.15) / 288.15))
```

**Top of Descent (3° descent path, geometry only)**

```
TOD_nm = ΔAlt / (tan(3°) × 6076 ft/nm) ≈ ΔAlt / 318.5
```

GS affects the *time* to descend, not the distance; the distance is purely geometric.

---

## Implementation

The project is a single `index.html` file — no npm, no bundler, no runtime dependencies. The wind vector diagram is drawn with the Canvas 2D API. Each calculator module is a self-contained block of DOM update logic with no shared state; modules read directly from input values and write results into output containers via `innerHTML`.

```
e6b/
├── index.html   ← all calculation logic, UI, styles, and canvas rendering
└── LICENSE
```

Calculation functions are pure — they take numbers, return numbers, and have no DOM side effects. The canvas respects dark/light theming by reading CSS custom properties via `getComputedStyle` on every redraw.

**Stack:** HTML5 · CSS custom properties · Vanilla JS · Canvas 2D API · GitHub Pages

---

## Parameters & Ranges

| Module | Inputs | Output |
|---|---|---|
| Wind Side | TC, TAS, wind dir/speed, var, dev | WCA, GS, TH, MH, CH |
| TSD | any two of dist / GS / time | the missing one |
| Fuel | flow rate, onboard, flight time | endurance, required, reserve |
| TAS | CAS, pressure alt, OAT | TAS, Mach, density alt, ISA dev |
| Altitude | indicated alt, QNH, OAT | pressure alt, density alt |
| Crosswind | runway hdg, wind dir/speed | headwind, crosswind (L/R) |
| TOD | alt to lose | distance to start descent |

---

## Disclaimer

> NOT FOR NAVIGATIONAL USE. All outputs must be verified with official charts and certified avionics before flight.

---

## Citation

```bibtex
@software{sari2026e6b,
    author  = {Kayra Sarı},
    year    = {2026},
    title   = {E6B Flight Computer},
    url     = {https://github.com/thekayrasari/e6b},
    license = {MIT}
}
```