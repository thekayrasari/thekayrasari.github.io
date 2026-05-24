---
title: "E6B Flight Computer & Vector Physics Simulator"
description: "A fully functional, browser-native E6B aviation navigation tool implementing the wind triangle, ISA atmospheric model, fuel planning, and a live Matter.js vector physics visualizer in vanilla JavaScript with zero runtime dependencies."
tags: ["aviation", "javascript", "matter-js", "physics-simulation", "navigation", "e6b"]
---

# E6B Flight Computer & Vector Physics Simulator

A fully functional aviation navigation tool built from first principles — wind triangle, ISA atmospheric model, fuel planning, and a live Matter.js physics visualizer in vanilla JavaScript with zero runtime dependencies.

---

**Table of Contents**

- [Overview](#overview)
- [Architecture](#architecture)
- [The Aviation Mathematics](#the-aviation-mathematics)
  - [Atmospheric Chain — Pressure → Density → TAS](#atmospheric-chain--pressure--density--tas)
  - [The Wind Triangle Solver](#the-wind-triangle-solver)
  - [Leg Performance — ETE & Fuel Burn](#leg-performance--ete--fuel-burn)
- [State Management & Reactivity](#state-management--reactivity)
- [Matter.js Physics Visualizer](#matterjs-physics-visualizer)
  - [Rigid Body & Force Model](#rigid-body--force-model)
  - [Bezel Compass & Whiz-Wheel Interaction](#bezel-compass--whiz-wheel-interaction)
  - [Wind Particle System](#wind-particle-system)
- [Test Suite](#test-suite)
- [Key Technical Decisions](#key-technical-decisions)
- [Running It](#running-it)
- [Repository](#repository)
- [Links](#links)

---

## Overview

**E6B Flight Computer** is a self-contained, zero-dependency aviation navigation tool that runs entirely in the browser. It reimplements the mechanical "whiz wheel" — the circular slide rule pilots have used since the 1940s — as a physically accurate computational model paired with a real-time vector physics canvas.

The tool solves three families of problems: the **wind triangle** (heading correction, ground speed, wind correction angle), **atmospheric conversions** (density altitude, true airspeed from IAS), and **fuel/time planning** (ETE, endurance). Every number the UI displays is derived from the same equations a pilot uses. Every slider change drives a live re-simulation of the aircraft's trajectory through air and wind.

The compute chain — atmospheric derivation, wind triangle solution, aerodynamic post-processing, and Matter.js physics — runs inside a single JavaScript event-driven loop with no WebGL, WebAssembly, or server-side compute. All state is held in a flat object; all math is in pure functions isolated from the DOM.

> This tool is designed for flight planning education and simulation, not certified navigation use. Always verify against certified avionics or approved publications before flight.

---

## Architecture

The project is a three-layer separation of concerns in plain JavaScript, with a thin Node.js development server for local use.

| File | Role |
|---|---|
| `calculator.js` | Pure aviation math — no DOM, no side effects. Dual-module compatible (Node + browser). |
| `app.js` | UI controller, state machine, input bindings, dual-control sliders, tab routing. |
| `visualizer.js` | Matter.js rigid body simulation, bezel compass rose, wind particle system, aircraft trail. |
| `server.js` | Zero-dependency Node.js HTTP server with MIME routing and directory traversal protection. |

The critical architectural decision is keeping `calculator.js` fully isolated from the browser environment. It exports an `E6BCalculator` object that works identically in Node.js (for the test suite) and in the browser (loaded as a global), making the math testable without a headless browser.

```js
// Dual-module compatibility pattern — bottom of calculator.js
if (typeof module !== 'undefined' && module.exports) {
  module.exports = E6BCalculator;   // Node.js / test runner
} else {
  window.E6BCalculator = E6BCalculator;  // Browser global
}
```

The `App` object in `app.js` holds all mutable UI state in a single flat object with nine numeric fields, making it trivial to inspect or serialize the application's condition at any point. There is no reducer, no proxy, no observable.

---

## The Aviation Mathematics

### Atmospheric Chain — Pressure → Density → TAS

The ISA (International Standard Atmosphere) defines how pressure and density decrease with altitude. The project implements the full chain from pressure altitude to true airspeed across three chained functions.

**Station Pressure** at a given pressure altitude uses the barometric formula from the ISA troposphere model:

```
P = 1013.25 × (1 − 6.8755856×10⁻⁶ × h)^5.25588   [hPa]
```

**Air Density** follows from the ideal gas law, with pressure in hPa and temperature in Kelvin:

```
ρ = (P × 100) / (287.05 × T_K)   [kg/m³]
```

**True Airspeed** is derived from IAS by correcting for the density ratio σ = ρ / ρ_SL, where ρ_SL = 1.225 kg/m³:

```
TAS = IAS / √σ
```

**Density Altitude** uses the standard approximation — 120 feet per degree Celsius of ISA deviation — accurate to within ~1% across the general aviation envelope:

```
DA = PA + 120 × (OAT − ISA_std)     where  ISA_std = 15 − 1.98 × (PA / 1000)
```

```javascript
calculateTAS(ias, pressureAlt, oat) {
  const press = this.calculatePressure(pressureAlt);
  const rho   = this.calculateDensity(press, oat);
  const sigma = rho / 1.225;         // density ratio
  if (sigma <= 0) return 0;
  return ias / Math.sqrt(sigma);
}
```

### The Wind Triangle Solver

The wind triangle is the core navigation problem: given a desired ground track (course), wind direction and speed, and aircraft true airspeed — find the required magnetic heading and the resulting ground speed. The solution involves the **Wind Correction Angle (WCA)**.

The three vectors form a closed triangle:

```
         WAYPOINT
        ↗ (course, GS)
       /  ↖ (wind, WS)
      /
GROMMET ——→ (heading, TAS)
```

The solver uses the sine rule to find WCA from the ratio of the wind's cross-component to TAS:

```
WCA = arcsin( WS × sin(W_dir − C) / TAS )
TH  = C + WCA
GS  = TAS × cos(WCA) − WS × cos(C − W_dir)
```

Headwind and crosswind components are decomposed separately from the wind angle relative to course:

```
HW = WS × cos(W_rel)    (positive = headwind, negative = tailwind)
XW = WS × sin(W_rel)    (positive = from right, negative = from left)
```

A critical edge case: when wind speed exceeds TAS, `|WS × sin(angle) / TAS| > 1` — the arcsine domain is violated and the flight is aerodynamically impossible on the desired course. The solver detects this, sets `impossible: true`, and the UI enters an error state that prevents dependent calculations from producing misleading output. Rather than clamping to a bogus number, the flag cascades through all dependent outputs, zeroing ground speed and suppressing fuel/ETE display entirely.

### Leg Performance — ETE & Fuel Burn

Given a leg distance, ground speed, and fuel flow:

```
t = D / GS   [hours]
F = t × FF   [gallons]
```

ETE is formatted to `HHh MMm SSs` with zero-padded minutes and seconds, matching the style used on most glass-cockpit EFBs. The function guards against GS = 0 by returning an indeterminate sentinel that cascades correctly to the impossible-wind UI state rather than displaying nonsensical values.

---

## State Management & Reactivity

Every input has a matching range slider, and both controls stay synchronized without recursive event loops. The solution is a single `updateVal()` closure per binding that writes both controls and state simultaneously, then calls `calculateAll()`:

```javascript
const updateVal = (val) => {
  let parsed = bind.isInt ? parseInt(val, 10) : parseFloat(val);
  if (isNaN(parsed)) return;

  // Clamp to declared input limits
  const min = parseFloat(numInput.min);
  const max = parseFloat(numInput.max);
  parsed = Math.max(min, Math.min(max, parsed));

  this.state[bind.stateKey] = parsed;
  numInput.value = parsed;      // sync number input
  sliderInput.value = parsed;   // sync range slider

  this.calculateAll();
};
```

Because both listeners call the same function, and that function directly writes to both DOM elements before they can fire their own events, there is no ping-pong between inputs. No debounce, no observable, no reactive store needed.

Atmospheric inputs (IAS, pressure altitude, OAT) trigger an additional `syncAtmosphericTAS()` pass before the main calculation, because TAS is derived from IAS and feeds the wind triangle. This cascaded ordering — atmospheric sync before wind triangle — is a manually implemented topological sort of the dependency graph, ensuring the wind result always uses the current computed TAS rather than a stale input value.

---

## Matter.js Physics Visualizer

### Rigid Body & Force Model

The second panel is a live physics canvas built on Matter.js. The aircraft is a rigid body defined from a triangular vertex set. The engine runs with gravity disabled — this is a 2D top-down vector plane, not a vertical flight model.

```javascript
const aircraftPath = [
  { x: 0,   y: -15 },  // Nose
  { x: 10,  y: 12  },  // Right wingtip
  { x: 0,   y: 6   },  // Tail slot
  { x: -10, y: 12  }   // Left wingtip
];

this.aircraft = Bodies.fromVertices(this.cx, this.cy, aircraftPath, {
  frictionAir: 0.03,    // Aerodynamic drag coefficient
  label: 'aircraft'
});
```

At each telemetry update, two frame-relative forces are applied to the aircraft body:

- **Thrust force** — oriented along the true heading angle (TH), scaled by TAS. Represents the aircraft's own airspeed through the airmass.
- **Wind drift force** — oriented along the wind direction, scaled by wind speed. Represents the airmass carrying the aircraft.

The vector sum produces the aircraft's net ground track, which converges toward the desired course when the WCA is correctly applied. The `forceScale` constant (0.00004) converts knot-scale values into Matter.js force units that produce visually meaningful trajectories without numeric overflow.

The visualizer is not a numerical integration of true 6-DOF flight dynamics. It is a real-time pedagogical illustration of vector addition — the same concept the physical whiz wheel encodes mechanically.

### Bezel Compass & Whiz-Wheel Interaction

The compass rose supports drag rotation — the user can spin the bezel like a physical E6B by clicking and dragging. When "Bezel Sync: Course" is active, the bezel angle is locked to the course value and dragging it writes back to the course input in Panel A. When sync is disabled, the bezel becomes a free-rotating reference tool for measuring relative bearings manually, independent of the computed values.

Bezel drag uses `Math.atan2` relative to the canvas center, handling the full 360° range correctly without quadrant special-cases:

```javascript
const angle = Math.atan2(
  e.clientY - rect.top  - this.cy,
  e.clientX - rect.left - this.cx
) * 180 / Math.PI;

const delta = angle - this.startDragAngle;
this.bezelAngle = (this.startBezelAngle + delta + 360) % 360;
```

### Wind Particle System

Wind is visualized as a particle system where each particle originates at a random canvas edge and travels in the wind direction at a speed proportional to wind speed. Particles carry a lifetime counter and fade out on expiry. The system is intentionally sparse — too many particles obscure the vector arrows that are the primary communication medium.

---

## Test Suite

The test suite in `test.js` runs in Node.js with no test framework — just a custom `assert()` function and a `testCase()` wrapper that catches thrown errors and tallies pass/fail. Zero dependencies, meaningful regression coverage on the most critical layer of the application.

```
=== Running E6B Flight Computer Tests ===
[PASS] Standard Conditions at Sea Level
[PASS] Density Altitude & TAS at 5,000 ft
[PASS] Direct Headwind
[PASS] Direct Tailwind
[PASS] Pure Crosswind from Right
[PASS] Leg Performance (ETE & Fuel)
[PASS] Strong Wind (Impossible Flight)

=== Test Results: 7 passed, 0 failed ===
```

The crosswind test is the most mathematically thorough — it verifies all five output fields of `solveWindTriangle()` simultaneously: WCA (arcsin derivation), heading (course + WCA mod 360), ground speed (TAS × cos(WCA) − wind component), and the decomposed headwind/crosswind values. Tolerances are tight: 0.1° for angles, 0.2 kts for speeds, to catch any regression in the trigonometric model.

The impossible-wind test verifies that `|WS × sin(angle) / TAS| > 1` is correctly detected (WS = 150 kts, TAS = 120 kts), `impossible` is flagged, and GS is capped at zero rather than producing a negative or undefined result.

---

## Key Technical Decisions

**No frontend framework.** A flight computer has a fixed, known input set and a deterministic output set. Framework overhead — VDOM diffing, reactivity graphs, build tooling — adds complexity with no benefit for a bounded domain like this.

**Zero-dependency server.** Node's built-in `http` and `fs` modules are sufficient for serving static files locally. No Express, no middleware, fewer supply-chain attack surfaces.

**Matter.js over raw Canvas.** Using a physics engine lets the aircraft respond physically to applied forces, producing natural drift and settling behavior rather than teleporting to computed positions. It also decouples the visualization from the math layer cleanly.

**Dual-module `calculator.js`.** Keeping the math testable in Node.js without a DOM environment means the most critical code — the aviation formulas — can be verified with fast, headless unit tests on every change.

**Flat state object.** Nine numeric fields. No normalization needed. Direct writes are readable and debuggable. The entire state serializes with a single `JSON.stringify(App.state)`.

**`impossible` flag propagation.** Rather than clamping to a bogus number, the flag cascades through all dependent outputs — zeroing ground speed and suppressing fuel/ETE rather than displaying aerodynamically nonsensical values.

---

## Running It

```bash
npm install     # only installs gh-pages for deployment
npm start       # dev server at http://localhost:3000
npm test        # aviation math test suite
npm run deploy  # publishes to GitHub Pages
```

---

## Repository

```
e6b/
├── index.html      # Application shell and UI markup
├── style.css       # Layout, HUD, panel cards
├── app.js          # UI controller, state, input bindings
├── calculator.js   # Pure aviation math (browser + Node)
├── visualizer.js   # Matter.js canvas + compass + particles
├── server.js       # Zero-dependency static file server
├── test.js         # Standalone unit test suite
└── package.json    # npm scripts: start, test, deploy
```

- Runtime dependencies: **0**
- Dev dependencies: **1** (gh-pages)
- Test cases: **7**
- Browser requirement: any modern browser with Canvas 2D API

---

## Links

- **Live Demo:** [thekayrasari.github.io/e6b](https://thekayrasari.github.io/e6b)
- **Author:** Kayra Sarı
- **Solver Core:** ISA atmospheric model + wind triangle — runs entirely client-side, no dependencies.

---