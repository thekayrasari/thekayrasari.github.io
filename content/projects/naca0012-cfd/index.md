---
title: "NACA 0012 CFD Simulator"
date: 2026-03-06
tags: ["aerodynamics", "CFD", "NACA 0012", "simulation", "thin airfoil theory"]
author: ["Kayra Sari"]
description: "A browser-based aerodynamic simulator for the NACA 0012 symmetric airfoil, computing lift, drag, and stall behavior across a full angle-of-attack sweep in real time. No dependencies, no build step — just HTML, CSS, and JavaScript."
summary: "An interactive CFD-style tool that models the NACA 0012 airfoil using a viscous-corrected empirical method, with Reynolds number sensitivity, hysteresis-aware stall detection, and a live C_L vs α polar plot."
---

> Live demo: [thekayrasari.github.io/naca0012-cfd](https://thekayrasari.github.io/naca0012-cfd/) · [Source on GitHub](https://github.com/thekayrasari/naca0012-cfd)

---

## Abstract

The NACA 0012 is one of the most extensively studied symmetric airfoil profiles in aeronautics, serving as a benchmark in both wind tunnel experiments and computational studies. This project is a self-contained, browser-based simulator that computes aerodynamic coefficients for the NACA 0012 across a user-defined range of angles of attack.

The simulation uses a viscous-corrected empirical model built on thin airfoil theory, incorporating Reynolds number effects and a hysteresis-aware stall onset model. All computation runs client-side with zero external dependencies — the page itself is the simulation.

---

## How It Works

Thin airfoil theory gives a lift slope of 2π per radian for symmetric profiles. This serves as the baseline, with viscous corrections applied to bring predictions into agreement with experimental data across the Reynolds number range of 0.5M to 6.0M.

Drag is modeled as the sum of a zero-lift parasitic component and an induced term that scales with C_L². Stall is triggered when α exceeds a Reynolds-number-dependent threshold, with hysteresis applied so the flow does not instantly reattach when α decreases back through the stall angle.

**Controls:**

- **Angle of attack** — swept from −20° to +25° in configurable steps
- **Reynolds number** — adjustable from 0.5M to 6.0M; higher Re delays stall onset
- **Sweep step** — 0.5° to 2.0°; finer steps give smoother polar curves

The C_L vs α polar updates live as the sweep runs, and readouts for C_L, C_D, and L/D refresh at each step.

---

## Key Parameters

**Airfoil geometry**

| Parameter | Value |
|---|---|
| Profile | NACA 0012 |
| Max thickness | 12% chord |
| Chord length | 1.00 m |
| Span | ∞ (2D analysis) |

**Simulation model**

| Parameter | Value |
|---|---|
| Method | Viscous empirical, calibrated to wind tunnel data |
| Flow regime | Incompressible |
| α range | −20° to +25° |
| Re range | 0.5M – 6.0M |
| Stall model | Re-dependent + hysteresis |

---

## Key Equations

**Lift coefficient (thin airfoil baseline)**

```
C_L = 2π × sin(α)
```

**Drag model**

```
C_D = C_D0 + k × C_L²
```

where `C_D0` is the zero-lift drag and `k` is the induced drag factor, both Re-dependent.

**Aerodynamic efficiency**

```
L/D = C_L / C_D
```

**Stall criterion**

```
α > α_stall(Re)   →   flow separation detected
```

Stall angle increases with Reynolds number as the boundary layer remains attached longer at higher Re.

---

## Implementation

The entire project is a single `index.html` file. The airfoil geometry renderer, simulation core, and polar chart are all written from scratch using the Canvas 2D API and vanilla JavaScript — no npm, no bundler, no runtime dependencies.

```
naca0012-cfd/
└── index.html   ← geometry renderer + sim engine + polar chart
```

**Stack:** HTML5 · CSS3 · Vanilla JS · Canvas 2D API · GitHub Pages

---

## Citation

```bibtex
@software{sari2026naca0012,
    author  = {Kayra Sarı},
    year    = {2026},
    title   = {NACA 0012 CFD Simulator},
    url     = {https://github.com/thekayrasari/naca0012-cfd},
    license = {MIT}
}
```