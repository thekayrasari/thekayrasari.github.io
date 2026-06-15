---
title: "CADES: Computational Aerodynamics Display & Evaluation System"
description: "A high-performance, interactive 2D aerodynamics simulator built with the Lattice Boltzmann Method (LBM) and D2Q9 lattice."
tags: ["aerodynamics", "cfd", "lbm", "simulation", "javascript", "physics"]
---

# CADES — Computational Aerodynamics Display & Evaluation System

The Computational Aerodynamics Display & Evaluation System (CADES) is a state-of-the-art, high-performance 2D aerodynamics simulator designed to run seamlessly in the browser. It implements the Lattice Boltzmann Method (LBM) using a D2Q9 lattice architecture to provide real-time fluid dynamics visualization. This platform is engineered specifically for exploring airflow behaviors around a NACA 0012 airfoil, providing instantaneous feedback on pressure distributions, lift coefficients, and wake turbulence.

## Mathematical and Physical Underpinnings

### The Lattice Boltzmann Method (LBM)
Traditional computational fluid dynamics (CFD) often relies on directly solving the Navier-Stokes equations macroscopically. In contrast, CADES uses the mesoscopic Lattice Boltzmann Method, which evolves particle distribution functions on a discrete lattice grid. The simulation specifically utilizes the **D2Q9 model** (2 Dimensions, 9 discrete velocity directions), which is optimal for 2D incompressible flows.

The collision and streaming processes are governed by the Bhatnagar-Gross-Krook (BGK) operator. However, to maintain numerical stability at higher Reynolds numbers, CADES integrates a **Smagorinsky Subgrid-Scale (SGS) turbulence model**. This dynamic closure model calculates the local strain rate tensor and adapts the relaxation time dynamically, adding artificial viscosity only where turbulent scales are too small for the grid to resolve. This approach prevents numerical blow-ups while preserving the macroscopic flow structures such as von Kármán vortex streets.

### Boundary Conditions
The fidelity of any CFD simulation depends heavily on its boundary conditions. CADES implements:
- **Zou-He Velocity Inlet:** Applied at the left boundary, this scheme analytically solves for unknown distribution functions to strictly enforce the prescribed velocity profile.
- **Extrapolation Outlet:** A zero-gradient condition at the right boundary allows vortical structures to exit the computational domain without causing non-physical pressure reflections.
- **Mid-Link Bounce-Back:** On the surface of the parametric NACA 0012 airfoil, incoming particle distributions are reflected back along their incoming paths. This strictly enforces the no-slip condition required for viscous boundary layers.

## Aerodynamic Performance Analysis

CADES computes aerodynamic coefficients entirely in real-time by integrating the localized pressure fields. 

### Pressure Coefficient (Cp)
The pressure field is extrapolated from the local fluid density ($p = c_s^2 \rho$). The solver samples the pressure normal to the airfoil surface across 40 chordwise stations to generate live $C_p$ curves. The formula utilized is:
$$C_p = \frac{p - p_\infty}{0.5 \cdot \rho_\infty \cdot u_0^2}$$
These distributions accurately capture the leading-edge stagnation point ($C_p = 1.0$) and the characteristic suction peak on the upper surface at positive angles of attack.

### Lift Coefficient (Cl) and Stall Detection
The vertical pressure force ($F_y$) is integrated numerically along the chord. The Lift Coefficient is then evaluated continuously. Furthermore, CADES includes a strict phenomenological stall detection heuristic, which triggers warnings when the angle of attack ($\alpha$) reaches or exceeds 12°. At these regimes, the user can visually confirm flow separation and the loss of the upper-surface suction peak.

## Software Architecture

The entire solver is built using highly optimized JavaScript, utilizing typed arrays (`Float32Array`) to eliminate garbage collection overhead and ensure JIT-compilation efficiency. The 150×75 grid (11,250 cells) executes its collision, streaming, and macroscopic recovery phases multiple times per animation frame, achieving real-time interaction natively.

Visual rendering is decoupled from the physics engine and utilizes dynamic color mapping (diverging colormaps for pressure and vorticity, heatmaps for velocity) scaled directly to high-DPI canvases. Additionally, a custom Lagrangian particle advection engine tracks up to 2,500 massless tracer particles, integrating their trajectories via bilinear interpolation of the discrete lattice velocity field.
