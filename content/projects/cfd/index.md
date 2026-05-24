---
title: "AeroDyn 2D: NACA 0012 Interactive CFD Wind Tunnel"
description: "A real-time, browser-native 2D CFD wind tunnel built on the Lattice Boltzmann Method (D2Q9) with a Smagorinsky subgrid-scale turbulence closure, simulating flow around a NACA 0012 airfoil entirely client-side."
tags: ["aerodynamics", "cfd", "lbm", "simulation", "javascript", "webgl"]
---

# AeroDyn 2D: NACA 0012 Interactive CFD Wind Tunnel

A real-time, browser-native 2D CFD wind tunnel powered by the Lattice Boltzmann Method.

---

**Table of Contents**

- [Overview](#overview)
- [Airfoil Geometry](#airfoil-geometry)
- [The Lattice Boltzmann Method](#the-lattice-boltzmann-method)
  - [D2Q9 Velocity Set](#d2q9-velocity-set)
  - [Collision: BGK with Smagorinsky SGS Closure](#collision-bgk-with-smagorinsky-sgs-closure)
  - [Streaming & Bounce-Back](#streaming--bounce-back)
  - [Boundary Conditions](#boundary-conditions)
- [Solver Architecture](#solver-architecture)
  - [Grid & State Representation](#grid--state-representation)
  - [Double-Buffering Strategy](#double-buffering-strategy)
  - [Numerical Stability](#numerical-stability)
- [Aerodynamics Post-Processing](#aerodynamics-post-processing)
  - [Pressure Coefficient Distribution](#pressure-coefficient-distribution)
  - [Lift Coefficient Estimation](#lift-coefficient-estimation)
  - [Stall Detection](#stall-detection)
- [Visualization Pipeline](#visualization-pipeline)
  - [Scalar Field Rendering via LUTs](#scalar-field-rendering-via-luts)
  - [Flow Tracing Particles](#flow-tracing-particles)
  - [Static Streamlines](#static-streamlines)
- [Interactive Controls](#interactive-controls)
- [Performance Characteristics](#performance-characteristics)
- [Known Limitations & Future Work](#known-limitations--future-work)
- [Links](#links)

---

## Overview

**AeroDyn 2D** is a self-contained, zero-dependency fluid dynamics simulator that runs entirely in the browser. It solves the 2D incompressible Navier-Stokes equations in the weakly compressible limit using the Lattice Boltzmann Method (LBM) on a structured D2Q9 lattice, with a NACA 0012 airfoil as the immersed solid boundary. The simulation exposes interactive control over angle of attack (AoA), Reynolds number, and inlet velocity, and provides real-time post-processing of the pressure coefficient (Cp) distribution, lift coefficient (Cl), and wake vorticity field.

The entire compute chain — geometry generation, collision, streaming, boundary enforcement, aerodynamic integration, and pixel rendering — executes synchronously inside a single JavaScript `requestAnimationFrame` loop, with no WebGL, WebAssembly, or server-side compute. All field arrays are allocated as typed `Float32Array` buffers to exploit JIT-friendly memory layout and minimize garbage collection pressure.

---

## Airfoil Geometry

The NACA 0012 is a symmetric four-digit series airfoil with a maximum thickness of 12% of chord. Its half-thickness distribution is defined analytically as:

```
y_t(x) = c · ( 0.5946895·√x̄ − 0.126·x̄ − 0.3516·x̄² + 0.2843·x̄³ − 0.1015·x̄⁴ )
```

where `x̄ = x/c` is the normalized chordwise coordinate and `c` is the chord length in lattice units (set to **40 lu**). This is the standard NACA modified-thickness form, yielding a closed trailing edge and a well-defined leading-edge radius.

The solid boundary mask is constructed by iterating over every lattice node and testing whether it falls within the airfoil profile in the airfoil's body frame. To accommodate a user-specified angle of attack `α`, each candidate node at global position `(x, y)` is first transformed into the airfoil frame by rotating its displacement vector from the leading edge through `−α`:

```javascript
const rx =  dx·cos(α) + dy·sin(α);
const ry = −dx·sin(α) + dy·cos(α);
```

A node is tagged as solid (`solid[idx] = 1`) if and only if `0 ≤ rx ≤ c` and `|ry| ≤ y_t(rx/c)`. Because this test is applied pixel-by-pixel on the integer lattice, the boundary is a staircase approximation of the smooth geometry — acceptable for the D2Q9 bounce-back scheme at moderate Reynolds numbers.

Whenever AoA changes, `updateSolidMask()` zeroes the entire mask array and recomputes it from scratch in O(Nx·Ny) time, which on a 150×75 grid (11 250 nodes) completes in under 0.5 ms and triggers a full solver reset to eliminate transient field corruption.

The airfoil leading edge is placed at lattice coordinates `(xc, yc) = (35, 37)`, leaving approximately 35 lattice units of upstream development length and 75 units of downstream wake capture length.

---

## The Lattice Boltzmann Method

LBM is a mesoscopic approach derived from the Boltzmann kinetic equation. Rather than solving the Navier-Stokes equations directly, it evolves discrete probability distribution functions `fᵢ(x, t)` — representing the number density of fictitious fluid particles streaming in direction `i` — on a regular Cartesian lattice. Macroscopic quantities are recovered as moments of these distributions.

### D2Q9 Velocity Set

The simulation uses the standard **D2Q9** model: two spatial dimensions, nine discrete velocity directions. The nine lattice velocities are:

```
Direction   (cx, cy)    Weight w
    0        ( 0,  0)    4/9
    1        (+1,  0)    1/9
    2        ( 0, +1)    1/9
    3        (−1,  0)    1/9
    4        ( 0, −1)    1/9
    5        (+1, +1)    1/36
    6        (−1, +1)    1/36
    7        (−1, −1)    1/36
    8        (+1, −1)    1/36
```

Macroscopic density ρ and velocity **u** are recovered from the zeroth and first moments:

```
ρ = Σᵢ fᵢ
ρu = Σᵢ cᵢ fᵢ
```

Pressure follows the isothermal equation of state: `p = ρ·cs²`, where the lattice speed of sound is `cs = 1/√3`. In this implementation, pressure is directly proportional to local density, so the `rho_grid` array serves as the pressure field for all post-processing computations.

### Collision: BGK with Smagorinsky SGS Closure

The standard single-relaxation-time (SRT) BGK collision operator relaxes the distribution functions toward the Maxwell-Boltzmann equilibrium:

```
fᵢ_eq = wᵢ·ρ·( 1 + 3(cᵢ·u) + 4.5(cᵢ·u)² − 1.5|u|² )
```

The post-collision state is:

```
fᵢ*(x, t) = fᵢ(x, t) + ω·( fᵢ_eq − fᵢ(x, t) )
```

where the global relaxation frequency is `ω = 1/τ` and `τ = 3ν + 0.5`, with kinematic viscosity `ν = u₀·c/Re`.

At moderate-to-high Reynolds numbers (Re > ~300), the plain BGK scheme becomes numerically unstable due to unresolved sub-lattice turbulent structures. To stabilize wake shedding and suppress `NaN` blow-ups, a **Smagorinsky subgrid-scale (SGS) LES model** is applied locally at every fluid node. This dynamically increases the effective relaxation time based on the magnitude of the non-equilibrium stress tensor.

The non-equilibrium part of each distribution is:

```
qᵢ = fᵢ − fᵢ_eq
```

The second-order non-equilibrium stress tensor components are assembled from these residuals:

```
Q_xx = q₁ + q₃ + q₅ + q₆ + q₇ + q₈
Q_yy = q₂ + q₄ + q₅ + q₆ + q₇ + q₈
Q_xy = q₅ − q₆ + q₇ − q₈

|Q| = √( Q_xx² + Q_yy² + 2·Q_xy² )
```

The locally adjusted relaxation time is then:

```
τ_local = τ + 0.5·( √(τ² + Cs²·18√2·|Q|/ρ) − τ )
```

with Smagorinsky constant `Cs = 0.15`. This is the standard LBM formulation of the Smagorinsky-Lilly SGS model, derived by Hou et al. (1994), ensuring that regions of high strain rate — particularly in the separated shear layers and vortex cores of the wake — receive additional numerical dissipation proportional to their local turbulent activity, without globally over-damping the solution.

### Streaming & Bounce-Back

After collision, post-collision distributions stream to neighboring nodes in their respective directions. For interior fluid nodes this is a simple array index shift:

```
fᵢ(x + cᵢ, t+1) ← fᵢ*(x, t)
```

When the target neighbor is a **solid node**, the standard **mid-link bounce-back** condition is applied: the outgoing distribution is reflected back into the originating node in the opposite direction. For example, if direction 1 (+x) streams into a solid, it is redirected back as direction 3 (−x):

```javascript
if (solid[n_idx]) f3_new[idx] = f1_star;
else              f1_new[n_idx] = f1_star;
```

This enforces a no-slip (zero velocity) condition on all solid surfaces to first-order accuracy in the lattice spacing. The bounce-back is integrated directly into the streaming loop to avoid a separate obstacle-handling pass.

### Boundary Conditions

**Inlet (x = 0): Zou-He Velocity BC**

The left boundary enforces a uniform inlet velocity `u₀` in the x-direction with zero transverse velocity. The Zou-He scheme analytically derives the unknown incoming distributions from the known macroscopic target state and the already-streamed outgoing distributions, ensuring exact enforcement of `ux = u₀` and `uy = 0` at every inlet node:

```
ρ_inlet = ( f₀ + f₂ + f₄ + 2(f₃ + f₆ + f₇) ) / (1 − u₀)
f₁ = f₃ + (2/3)·ρ·u₀
f₅ = f₇ − 0.5(f₂ − f₄) + (1/6)·ρ·u₀
f₈ = f₆ + 0.5(f₂ − f₄) + (1/6)·ρ·u₀
```

This is second-order accurate and avoids spurious pressure waves that would arise from simpler fixed-distribution inlet conditions.

**Outlet (x = Nx−1): Zero-Gradient Extrapolation**

The right boundary uses a first-order extrapolation (copy) scheme: all nine distributions at the outlet plane are set equal to their values one node upstream. This is a convective outlet condition that allows vortical structures to exit the domain without significant reflection, while keeping the implementation trivial.

**Top/Bottom (y = 0, y = Ny−1): Periodic**

Vertical boundaries wrap periodically. Array index arithmetic uses modular wrapping `(y + Ny) % Ny` throughout the streaming and particle-update loops, effectively simulating an infinitely tall wind tunnel cross-section. At low angles of attack this is a reasonable approximation for a tunnel with blockage ratio `chord/Ny ≈ 0.53`, though at high AoA the blockage effect is non-negligible.

---

## Solver Architecture

### Grid & State Representation

The lattice occupies a **150 × 75** node grid (Nx × Ny = 11 250 cells). Nine distribution function arrays are maintained — one per lattice velocity — each as a flat `Float32Array` of length `numCells = 11 250`. Row-major layout is used: node `(x, y)` maps to index `y·Nx + x`.

Three additional macroscopic arrays (`u_x_grid`, `u_y_grid`, `rho_grid`) are updated every solver step from the zeroth and first moments of the distributions. These are the arrays consumed by all visualization and post-processing code.

A `Uint8Array solid[]` encodes the solid mask: `1` for airfoil interior nodes, `0` for fluid nodes. Solid nodes are skipped entirely in the collision loop, reducing unnecessary computation.

### Double-Buffering Strategy

The solver maintains two complete sets of distribution arrays: `f0…f8` (current) and `f0_new…f8_new` (next). Collision and streaming write exclusively into the `_new` arrays, preventing in-place overwriting that would corrupt streaming from already-updated neighbors. After each step, the two sets are swapped by pointer exchange (array reference reassignment in JavaScript), a zero-copy O(1) operation:

```javascript
let temp = f0; f0 = f0_new; f0_new = temp;
```

This guarantees consistent read access across all nine streaming directions without any spatial padding or ghost cells.

### Numerical Stability

Several hardening measures prevent floating-point blow-up at high Reynolds numbers:

- **Density clamping:** `ρ ∈ [0.2, 2.0]` — prevents negative densities from producing imaginary velocities.
- **Velocity clamping:** `|ux|, |uy| ≤ 0.25` — enforces sub-sonic Mach number `Ma = u/cs < 0.43`, keeping the weakly-compressible LBM approximation valid.
- **NaN detection:** After every solver step, `f0[0]` is checked for `NaN`. If detected, the simulation is automatically reset to a clean equilibrium state, preventing the browser from hanging.
- **Smagorinsky dissipation:** As described above, locally amplified viscosity suppresses under-resolved turbulent structures.

---

## Aerodynamics Post-Processing

### Pressure Coefficient Distribution

The pressure coefficient at a surface point is defined as:

```
Cp(x) = (p(x) − p∞) / (0.5·ρ∞·u₀²)
```

In LBM units, `p = ρ/3`, so `p∞ = ρ∞/3 = 1/3` (using reference density `ρ∞ = 1`). The local pressure is sampled one lattice unit outward from the airfoil surface, along the surface normal, at `chord = 40` chordwise stations.

For each chordwise station `i`, the upper and lower surface sample points are computed in the rotated airfoil frame and then projected back to global lattice coordinates:

```
Upper: (gxTop, gyTop) = (xc + dxTop + sinA·1.2, yc + dyTop − cosA·1.2)
Lower: (gxBot, gyBot) = (xc + dxBot − sinA·1.2, yc + dyBot + cosA·1.2)
```

The 1.2-unit normal offset avoids sampling inside the staircase boundary. The resulting chordwise Cp curves — upper surface in slate blue, lower surface in red — are plotted live on a canvas graph with axes spanning Cp ∈ [−1.5, 3.0], following the aerodynamics convention of plotting −Cp upward (suction above the chord line).

### Lift Coefficient Estimation

Lift is estimated by integrating the pressure difference between lower and upper surfaces across the chord:

```
Fy = Σᵢ ( p_bottom(i) − p_top(i) )
Cl = Fy / ( 0.5·ρ∞·u₀²·c )
```

This is a straightforward application of the Kutta-Joukowski theorem in discretized form. The result is updated and displayed in the live telemetry panel every frame. Note that this is a pressure-only lift estimate and does not include the viscous drag contribution; for a symmetric airfoil at zero AoA, `Cl` should converge to zero as the simulation reaches steady state, which serves as a useful sanity check.

### Stall Detection

The simulator flags a stall condition when `|AoA| ≥ 12°`. This threshold is consistent with the empirical stall angle of the NACA 0012 at low-to-moderate Reynolds numbers (typically 12°–14° depending on Re). At stall, the Cp plot shows loss of suction peak on the upper surface and the telemetry panel highlights the lift coefficient box with a red warning indicator. At the lattice Reynolds numbers available in this simulator (Re ≤ 800), the separation is diffuse and the Smagorinsky model helps prevent the solver from blowing up in the fully separated wake.

---

## Visualization Pipeline

### Scalar Field Rendering via LUTs

Rendering the 150×75 scalar field at interactive frame rates without GPU compute requires aggressive optimization. The approach used is an **offscreen pixel buffer with precomputed color lookup tables (LUTs)**.

Three 256-entry `Uint32Array` LUTs are precomputed: one each for velocity magnitude, pressure, and vorticity. Each entry stores a packed 32-bit RGBA pixel in little-endian ABGR format (as required by `ImageData`). The mapping is computed once on initialization (and when the theme changes), so the inner render loop performs only a single array lookup per cell:

```javascript
const speed = Math.sqrt(ux² + uy²);
const val   = Math.min(255, Math.round((speed / 0.12) * 255));
offBuf32[i] = velocityLUT[val];
```

The 150×75 pixel buffer is written to an offscreen `ImageData`, uploaded to an offscreen canvas via `putImageData`, and then scaled to the display canvas resolution using `drawImage`. The `image-rendering: pixelated` CSS property ensures nearest-neighbor upscaling, preserving the lattice structure without blurring.

Three background fields are available:

- **Velocity magnitude** — monochromatic cool-to-hot heatmap (light theme) or five-band jet colormap (dark theme), normalized to a reference speed of `0.12 lu/step`.
- **Pressure** — diverging blue-grey-red colormap, mapping `ρ ∈ [0.93, 1.07]` to the full 0–255 range. This visualizes the stagnation pressure region at the leading edge and the suction region over the upper surface.
- **Vorticity** — diverging cyan-neutral-orange colormap, mapping curl `ω ∈ [−0.05, 0.05]` computed via central finite differences. This is the most diagnostic field for identifying vortex shedding, separated shear layers, and the von Kármán wake.

Vorticity at node `(x, y)` is approximated as:

```
ω(x, y) = ∂uy/∂x − ∂ux/∂y
         ≈ [uy(x+1,y) − uy(x−1,y)] / 2 − [ux(x,y+1) − ux(x,y−1)] / 2
```

with one-sided differences applied at domain boundaries.

### Flow Tracing Particles

Up to **2 500** massless Lagrangian tracer particles are advected through the velocity field each frame. Each particle stores its continuous position `(p_x, p_y)` and a scalar life value in the range `[0, 1]`. At each timestep, the local fluid velocity at the particle's sub-cell position is recovered via bilinear interpolation over the four surrounding lattice nodes:

```
u(p) = (1−tx)(1−ty)·u(x₀,y₀) + tx(1−ty)·u(x₁,y₀)
     + (1−tx)ty·u(x₀,y₁) + tx·ty·u(x₁,y₁)
```

The particle is then displaced by `u·dt` with `dt = 1` lattice time step. Particles are recycled back to the inlet (`p_x = 0`, `p_y = random`) when they exit the domain, strike the solid boundary, or exhaust their life budget (which decays at a randomized rate to produce a natural-looking temporal distribution of streaks). The user can manually inject ink bursts at any canvas location by clicking and dragging, overwriting particle positions with a cluster of 15 new particles centered on the cursor.

### Static Streamlines

As an alternative to particles, a static streamline overlay traces 22 equally-spaced streamlines across the inlet height. Each streamline is integrated forward using the current instantaneous velocity field with Euler stepping (`dt = 0.8`) for up to 260 integration steps, terminated on domain exit or solid contact. This provides a cleaner, snapshot-style picture of the flow topology — useful for identifying stagnation points and separation bubbles — at the cost of not showing time-dependent shedding dynamics.

---

## Interactive Controls

| Parameter | Range | Effect |
|---|---|---|
| Angle of Attack | −20° to +20° | Rotates airfoil boundary mask; regenerates solid field instantly |
| Reynolds Number | 50 to 800 | Modifies kinematic viscosity: `ν = u₀·c/Re`, adjusting `τ` accordingly |
| Inlet Velocity u₀ | 0.010 to 0.080 | Scales flow speed; also affects Re and τ |
| Max Particles | 200 to 2500 | Controls tracer particle budget |
| Background Field | velocity / pressure / vorticity / none | Selects scalar field for heatmap rendering |
| Visual Overlay | particles / streamlines / none | Selects Lagrangian or Eulerian flow visualization |
| Theme | Light / Dark | Switches color palette for all LUTs and overlays |

All parameter changes take effect on the next solver step without resetting the simulation, except for AoA changes which trigger an immediate solid mask rebuild. The **Reset Wind Tunnel** button re-initializes all distribution functions to the equilibrium state corresponding to uniform flow at the current `u₀`, effectively starting the simulation from rest.

**Step Frame** advances the solver by exactly one collision-streaming cycle when the simulation is paused, enabling frame-by-frame inspection of transient phenomena.

---

## Performance Characteristics

The solver runs **3 collision-streaming steps per animation frame** (`stepsPerFrame = 3`) to accelerate the initial transient and reach a statistically steady wake without sacrificing interactivity. On a modern mid-range CPU (e.g., an Intel Core i5 at ~3 GHz), the simulation sustains approximately 55–65 FPS at the default particle count of 1 200, delivering an effective 165–195 LBM time steps per wall-clock second.

The dominant cost is the inner collision loop: `Nx·Ny = 11 250` iterations per step, each performing ~60 floating-point operations (equilibrium computation, Smagorinsky correction, 9-direction streaming). The particle update loop (`maxParticles` bilinear interpolations) is a secondary cost that scales linearly with particle count. LUT-based pixel rendering avoids any per-pixel color computation at draw time, keeping the render pass to effectively a memset followed by a single canvas upload.

All typed arrays are allocated once at startup. No heap allocations occur in the hot path, which keeps the garbage collector quiescent during steady-state operation.

---

## Known Limitations & Future Work

- **Staircase boundary:** The solid mask is pixelated at the lattice scale. At `chord = 40 lu`, the leading-edge radius is only ~2–3 nodes wide, introducing first-order geometric error. Curved boundary schemes (e.g., Bouzidi interpolated bounce-back) would improve accuracy.
- **2D only:** Real airfoil flows are three-dimensional, especially in the separated wake and at stall. Span-averaged forces from 2D LBM consistently over-predict lift near stall.
- **Blockage effect:** The periodic top/bottom boundaries with a chord/tunnel-height ratio of ~0.53 introduce non-negligible confinement effects at high AoA, artificially elevating lift.
- **Low Re range:** The solver is tuned for Re ≤ 800 (roughly the upper limit where single-GPU browser LBM without WebGL compute remains stable). Real NACA 0012 applications operate at Re = 10⁶–10⁷.
- **Pressure-only Cl:** Skin friction drag is not computed. A complete force balance would require integrating the viscous stress tensor along the surface, which requires accurate velocity gradients at the staircase boundary.
- **Single-core JS:** The entire solver runs on the main thread. A natural next step is offloading the collision-streaming loop to a Web Worker, freeing the UI thread for smooth interaction even at larger grid sizes.

Possible extensions include multi-relaxation-time (MRT) collision to improve isotropy, a WebGL compute shader port for GPU-accelerated lattice operations, and support for additional NACA four-digit profiles via parametric geometry input.

---

## Links

- **Live Demo:** [thekayrasari.github.io/naca0012](https://thekayrasari.github.io/naca0012) *(or wherever hosted)*
- **Author:** Kayra Sari
- **Solver Core:** LBM D2Q9 + Smagorinsky SGS — runs entirely client-side, no dependencies.

---