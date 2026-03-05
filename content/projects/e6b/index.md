---
title: "The E6-B Flight Computer: A Deep Dive into Building a High-Fidelity Aviation Calculator in React"
date: 2026-03-05
tags: ["react", "svg", "interactive design", "web development", "aviation", "mathematics", "touch events", "responsive design"]
author: ["Kayra Sari"]
description: "A comprehensive technical exploration of building a pixel-perfect digital recreation of the classic E6-B aviation flight computer using React 19, Vite 5, and pure SVG graphics. Covers mathematical foundations, interactive mechanics, and responsive design patterns."
summary: "This technical deep-dive examines the implementation of a fully interactive, responsive E6-B flight computer web application. Discover how logarithmic mathematical principles, SVG graphics, React state management, and advanced touch interactions combine to faithfully recreate a beloved analog instrument for the modern web."
cover:
    image: ""
    alt: "E6-B Flight Computer Web Application"
    relative: false
editPost:
    URL: "https://github.com/thekayrasari/e6b"
    Text: "GitHub Repository"
---

## Introduction

The E6-B flight computer is one of aviation's most iconic instruments. For over 80 years, pilots have relied on this circular slide rule to perform critical calculations in the cockpit: distance-to-climb, fuel consumption, wind correction angles, and ground speed conversions. Despite the advent of glass cockpits and GPS navigation, the E6-B remains a staple in flight training and serves as a mechanical embodiment of logarithmic mathematical principles.

This article explores a modern web-based implementation of the E6-B: a fully interactive, responsive flight computer built with React 19, Vite 5, and pure SVG graphics. We'll examine the mathematical foundations, React architecture, interactive mechanics, and design decisions that went into creating a pixel-perfect digital recreation of this analog classic.

---

## Table of Contents

1. [Historical Context & Design Philosophy](#historical-context--design-philosophy)
2. [Project Architecture](#project-architecture)
3. [Mathematical Foundations](#mathematical-foundations)
4. [SVG Implementation](#svg-implementation)
5. [React State Management & Hooks](#react-state-management--hooks)
6. [Interactive Mechanics](#interactive-mechanics)
7. [Responsive Design System](#responsive-design-system)
8. [Performance Optimization](#performance-optimization)
9. [Design Tokens & Visual Language](#design-tokens--visual-language)
10. [Accessibility & UX Considerations](#accessibility--ux-considerations)
11. [Future Enhancements & Extensibility](#future-enhancements--extensibility)
12. [Deployment & Distribution](#deployment--distribution)

---

## Historical Context & Design Philosophy

### The E6-B in Aviation History

The E6-B was invented in 1938 by Philip Dalton, a American pilot and engineer. The name itself is somewhat mundane—it was originally designated as "model C," but Dalton's next iteration became the "E-6-B." The instrument combines two computational surfaces:

- **The calculator side:** A logarithmic slide rule for distance-rate-time calculations and altitude/airspeed corrections
- **The wind side:** A vector triangle grid for computing wind correction angles and true airspeed

For decades, the E6-B was the primary computational device for flight planning and in-flight problem solving. While electronic calculators and modern avionics have reduced its reliance, it remains:

- A required tool for pilot certification
- A backup instrument in case of electrical failure
- A training vehicle for understanding aviation mathematics
- A symbol of analog problem-solving elegance

### Design Philosophy for This Implementation

This web-based E6-B maintains strict fidelity to the classic analog design while enhancing it with digital affordances:

1. **Analog Authenticity:** The visual layout, scale arrangements, and computational logic mirror the physical instrument exactly. Users familiar with the traditional E6-B experience immediate recognition.

2. **Digital Enhancement:** Where appropriate, the web version adds modern conveniences—precise numeric readouts, real-time calculations, persistent state, and instant feedback.

3. **Platform Agnosticism:** The application runs identically on desktop browsers, tablets, and smartphones. Touch events and mouse events are treated as first-class input methods.

4. **Zero Dependencies:** Beyond React and Vite, the entire UI is rendered via inline SVG and CSS-in-JavaScript. No external graphics libraries, icon fonts, or image assets are required. This minimizes the build footprint and shipping complexity.

5. **Brand Neutrality:** All manufacturer branding (ASA, Jeppesen) has been removed, making the tool suitable for white-label distribution or educational use without licensing concerns.

---

## Project Architecture

### Technology Stack

```
┌─────────────────────────────────────────────┐
│         React 19 + Vite 5                   │
├─────────────────────────────────────────────┤
│  ┌───────────────────────────────────────┐  │
│  │    React Hooks & Functional Components│  │
│  │  - useState: rotation, wind angle     │  │
│  │  - useRef: SVG DOM references        │  │
│  │  - useCallback: event handlers       │  │
│  │  - useEffect: lifecycle & time tick  │  │
│  └───────────────────────────────────────┘  │
├─────────────────────────────────────────────┤
│  ┌───────────────────────────────────────┐  │
│  │   SVG Rendering Layer                │  │
│  │  - 400x400 viewBox coordinate system │  │
│  │  - Radial & linear gradients         │  │
│  │  - Drop shadow filters & glow FX     │  │
│  │  - Text with rotation transforms     │  │
│  └───────────────────────────────────────┘  │
├─────────────────────────────────────────────┤
│  ┌───────────────────────────────────────┐  │
│  │   CSS-in-JS Design System             │  │
│  │  - Global style injection             │  │
│  │  - Responsive media queries           │  │
│  │  - Design tokens (colors, fonts)      │  │
│  │  - Animations & transitions           │  │
│  └───────────────────────────────────────┘  │
├─────────────────────────────────────────────┤
│     DOM: HTML + Inline Styles              │
└─────────────────────────────────────────────┘
```

### File Structure

```
e6b/
├── src/
│   ├── e6b.jsx              # Main component (820 lines, monolithic)
│   ├── App.jsx              # App wrapper
│   ├── main.jsx             # React entry point
│   ├── App.css              # App-level styles
│   ├── index.css            # Global resets
│   └── assets/              # Static files for build
├── public/                  # Static assets (favicon, etc.)
├── vite.config.js           # Vite bundler configuration
├── eslint.config.js         # Linting rules
├── package.json             # Dependencies & scripts
└── index.html               # HTML entry point
```

### Component Architecture

The application is currently a **single monolithic component** (`E6B`) containing:

1. **Global Styles & Design Tokens** – Constants defined at module level
2. **Utility Functions** – Mathematical helpers, coordinate transforms
3. **SVG Definition Components** – Reusable SVG filter and gradient definitions
4. **Sub-components** – Smaller rendering functions:
   - `Defs()` – SVG definitions
   - `ScaleMarks()` – Logarithmic scale marks
   - `RateArrow()` – Rate index pointer
   - `Windows()` – Dynamic data windows
   - `Hub()` – Central hub geometry
   - `CalcSVG()` – Calculator dial SVG
   - `WindSide()` – Wind triangle grid
   - `Badge()` – UI badge component
   - `ReadoutPanel()` – Calculation results display
   - `WindPanel()` – Instructions panel
   - `StatusBar()` – Tips & hints bar

5. **Root Component** – Main `E6B()` with state, effects, and event handlers

**Architectural Note:** While functional—and ideal for educational purposes—this monolithic approach could be refactored by extracting sub-components into separate files, particularly `CalcSVG`, `WindSide`, and the data panels. This would improve maintainability for teams or larger feature additions.

---

## Mathematical Foundations

### Logarithmic Scale Principle

The E6-B's power derives from logarithmic scale mapping. By mapping numbers to angles via the logarithm function, the instrument transforms multiplication and division into simple addition and subtraction of angles.

#### Formula: Angle from Value

For the E6-B's outer scale (nautical miles, knots, statute miles), values from 10 to 99 are mapped to angles using:

```
angle = (log₁₀(value) - 1) × 360°
```

**Why this formula?**

- `log₁₀(10) = 1`, so: `(1 - 1) × 360 = 0°` (start of scale)
- `log₁₀(100) = 2`, so: `(2 - 1) × 360 = 360°` (one full rotation = end of scale)
- `log₁₀(60) ≈ 1.778`, so: `(1.778 - 1) × 360 ≈ 280°`

This creates a non-linear scale where small numbers (10-20) occupy more angular space, and large numbers (80-99) are compressed. This mirrored the physical E6-B's construction.

#### Code Implementation

```javascript
const logA = v => (Math.log10(v) - 1) * 360;
```

This single line is the mathematical heart of the application.

### Polar Coordinate System

The E6-B uses a circular canvas centered at `(200, 200)` in a 400×400 SVG viewBox. All markings, labels, and pointers are positioned using polar coordinates, then converted to Cartesian coordinates for SVG rendering.

#### Conversion Functions

```javascript
const CX = 200, CY = 200;  // Circle center

const toRad = d => d * Math.PI / 180;   // degrees to radians
const toDeg = r => r * 180 / Math.PI;   // radians to degrees

function polar(r, deg) {
  const a = toRad(deg);
  // Standard math: y increases upward; SVG: y increases downward
  // Use sin for x (horizontal), -cos for y (inverted vertical)
  return [
    +(CX + r * Math.sin(a)).toFixed(3),
    +(CY - r * Math.cos(a)).toFixed(3)
  ];
}
```

The `polar()` function is called thousand of times during rendering:

```javascript
const [x1,y1] = polar(outerR, angle);     // Mark line start
const [x2,y2] = polar(outerR-10, angle);  // Mark line end
const [lx,ly] = polar(170, angle);        // Label position
```

**Precision:** Coordinates are rounded to 3 decimal places to keep SVG markup compact while maintaining visual precision.

### Rate Calculations

The **rate index** on the calculator side represents how quickly distance is being covered. The rate calculation dynamically updates based on the rotation of the inner wheel:

```javascript
const rate = (() => {
  const a = ((logA(60) + rot) % 360 + 360) % 360;
  return Math.pow(10, a / 360 + 1);
})();
```

Breaking this down:

1. `logA(60)` maps 60 knots to an angle
2. `+ rot` adds the current wheel rotation
3. `% 360` normalizes to 0-360 range (handles wrap-around)
4. `+ 360) % 360` ensures positive result
5. `Math.pow(10, ...)` inverts the logarithm: from angle back to value

**Real-world Application:** If the inner wheel is rotated such that the "60 kt" mark aligns with "1 hour," the rate shows as a large value (fast travel). Rotating to align with "10 minutes" shows slower travel.

### Wind Triangle Vector Math

The wind triangle solves the classic pilot problem:

```
True Wind Vector
      ↓
  ┌────────────┐
  │   TRUE     │  True Airspeed (TAS)
  │ AIRSPEED   │  + Wind Vector
  │            │  = Ground Speed
  └────────────┘
```

When a pilot marks a dot on the wind grid:

```javascript
const wSpd = windDot ? Math.round(Math.sqrt(
  (windDot.x - CX)**2 + (windDot.y - CY)**2
)) : null;
```

The distance from the grid center to the marked dot represents wind speed in knots (on a 1:1 scale where each pixel ≈ 1 knot at standard zoom).

---

## SVG Implementation

### Coordinate System & Viewport

The E6-B is rendered in a 400×400 SVG with `viewBox="0 0 400 400"`. This allows the SVG to scale infinitely without loss of quality:

```jsx
<svg viewBox="0 0 400 400"
  style={{width:"100%",height:"100%",display:"block"}}
>
  {/* Content scales proportionally */}
</svg>
```

The large viewBox size (relative to typical web graphics) provides:
- **Precision:** Stroke widths of 0.5–1.5 units remain crisp at any zoom level
- **Scalability:** Zoom to desktop (500px) or mobile (280px) without pixelation
- **Touch-friendliness:** Large hit areas even on small screens

### Gradients & Lighting

SVG gradients create the **illusion of depth** and physical material:

#### Radial Gradients (3D Bezel Effect)

```jsx
<radialGradient id="bezel" cx="40%" cy="35%">
  <stop offset="0%"   stopColor="#1c2333"/>
  <stop offset="60%"  stopColor="#111825"/>
  <stop offset="100%" stopColor="#07090f"/>
</radialGradient>
```

- Light highlight at top-left (`cx="40%"`, `cy="35%"`)
- Gradually darkens outward → simulates curved glass and shadow
- Creates a "depth and curvature" visual that a flat gradient cannot

Applied to the main bezel circle:

```jsx
<circle cx={CX} cy={CY} r={197} fill="url(#bezel)"/>
```

#### Linear Gradients (Surface Details)

```jsx
<linearGradient id="windowGlass" x1="0%" y1="0%" x2="100%" y2="100%">
  <stop offset="0%"   stopColor="#1a3a5c" stopOpacity="0.95"/>
  <stop offset="100%" stopColor="#0d1f38" stopOpacity="0.98"/>
</linearGradient>
```

Diagonal sweeps (top-left to bottom-right) simulate light catching glass surfaces. Used for the dynamic data windows showing altitude and fuel.

### Filters: Glow & Drop Shadow

#### Drop Shadow Filter

```jsx
<filter id="ds">
  <feDropShadow dx="0" dy="12" stdDeviation="26" 
    floodColor="#000" floodOpacity="0.92"/>
</filter>
```

Applied to the main bezel, this creates a **ground shadow** beneath the instrument, anchoring it spatially and separating it from the background.

#### Glow Filters

```jsx
<filter id="glow" x="-8%" y="-8%" width="116%" height="116%">
  <feDropShadow dx="0" dy="0" stdDeviation="3.5" 
    floodColor="#4da6ff" floodOpacity="0.55"/>
</filter>

<filter id="greenGlow" x="-8%" y="-8%" width="116%" height="116%">
  <feDropShadow dx="0" dy="0" stdDeviation="3" 
    floodColor="#4ade80" floodOpacity="0.5"/>
</filter>
```

The glow inherits the **system's accent color** (blue for calculator, green for wind). When applied to pointers and markings, the glow:
- Enhances readability in low-light contexts
- Signals interactivity and state change
- Creates a "luminous instrument" aesthetic

**Technical note:** The expanded bounding box (`x="-8%"... width="116%"`) prevents the blur from being clipped by the element's bounds.

### Scale Marks Generation

The outer scale must display 90 values (10 through 99) with varying mark weights:

```javascript
function mkMarks() {
  const out = [];
  for (let v = 10; v <= 99; v++) {
    const a = logA(v);
    const t10 = v % 10 === 0, t5 = v % 5 === 0, t2 = v % 2 === 0;
    let tk, lb, fs;
    if      (t10)         { tk=14; lb=`${v}`; fs=9.5; }
    else if (v<=20)       { tk=9;  lb=`${v}`; fs=7;   }
    else if (v<=30 && t2) { tk=7;  lb=`${v}`; fs=6.5; }
    else if (t5)          { tk=10; lb=`${v}`; fs=7.5; }
    else if (t2)          { tk=5;  lb='';     fs=0;   }
    else                  { tk=3;  lb='';     fs=0;   }
    out.push({ v, a, tk, lb, fs, bold: t10 });
  }
  return out;
}
const MARKS = mkMarks();
```

This strategy creates a **visual hierarchy**:

| Condition | Tick Length | Label | Size | Purpose |
|-----------|-------------|-------|------|---------|
| v % 10 = 0 (10, 20, ...) | 14px | Yes | 9.5px | **Major marks** – easy primary reference |
| 10 ≤ v ≤ 20 (any) | 9px | Yes | 7px | **Secondary marks** – detailed readings when needed |
| 20 < v ≤ 30 & even | 7px | Yes | 6.5px | **Tertiary marks** – for intermediate values |
| v % 5 = 0 | 10px | No | — | **Quinary marks** – visual guide without clutter |
| v % 2 = 0 | 5px | No | — | **Minor guides** |
| Odd, > 30 | 3px | No | — | **Base guides** |

Rendering each mark:

```jsx
function ScaleMarks({ outerR, color, accentColor, labelR }) {
  const numR = labelR ?? outerR - 8;
  return MARKS.map(({ v, a, tk, lb, fs, bold }) => {
    const [x1,y1] = polar(outerR-1, a);
    const [x2,y2] = polar(outerR-1-tk, a);
    const [lx,ly] = polar(numR-tk, a);
    const c = bold ? (accentColor ?? color) : color;
    return (
      <g key={v}>
        <line x1={x1} y1={y1} x2={x2} y2={y2}
          stroke={c} strokeWidth={bold?1.6:tk>=9?1.1:tk>=5?0.8:0.5}
          opacity={bold?1:0.7}/>
        {lb && (
          <text x={lx} y={ly} textAnchor="middle" 
            dominantBaseline="middle"
            fontSize={fs} fontFamily="'Fira Code',monospace"
            fontWeight={bold?"700":"600"} 
            fill={c} opacity={bold?1:0.88}
            transform={`rotate(${a},${lx},${ly})`}>
            {lb}
          </text>
        )}
      </g>
    );
  });
}
```

**Key SVG techniques:**

1. **Per-element rotation:** `transform={rotate(${a},${lx},${ly})}` rotates text to align with the scale while maintaining readability
2. **Visual emphasis:** Bold marks use thicker lines, brighter colors, and larger fonts
3. **Opacity layering:** Non-primary marks use reduced opacity (0.7) to visually recede
4. **Responsive typography:** Text size adapts to density (9.5px for major marks, 6.5px for tertiary)

---

## React State Management & Hooks

### State Variables

The root component manages six pieces of state, each driving a specific interaction:

```javascript
const [side, setSide]             = useState("calc");      // "calc" | "wind"
const [rot, setRot]               = useState(0);           // 0–360°, rotation of inner wheel
const [windAngle, setWindAngle]   = useState(0);           // 0–360°, azimuth heading
const [windOffset, setWindOffset] = useState(0);           // -130 to +130, grid slide position
const [windDot, setWindDot]       = useState(null);        // {x, y} or null, marked wind vector
const [trans, setTrans]           = useState(false);       // animation flag for flip
const [utc, setUtc]               = useState("--:--:--Z"); // HH:MM:SSZ current UTC time
```

**Design rationale:**

- **Minimal state:** Each value directly affects rendering and cannot be computed from other state. Derived values (like `rate` or `wSpd`) are computed on-demand.
- **Boolean flags:** `trans` is a transient animation flag—set to true during flip, then false after 200ms—triggering a CSS scale animation.
- **No object nesting:** Flat structure avoids deep comparison issues; each state update is atomic.

### useEffect: Global Style Injection

```javascript
useEffect(() => {
  const tag = document.createElement("style");
  tag.innerHTML = GLOBAL_STYLE;
  document.head.appendChild(tag);
  return () => document.head.removeChild(tag);
}, []);
```

**Purpose:** Inject the massive `GLOBAL_STYLE` constant (containing media queries, animations, and design tokens) once at mount. The cleanup function removes it on unmount.

**Why not just use `<style>` in JSX?** Keeping styles in a constant allows:
- Easy version control and review
- IDE syntax highlighting in template literals
- Logical separation from component logic

### useEffect: UTC Clock

```javascript
useEffect(() => {
  const tick = () => setUtc(new Date().toISOString().slice(11,19)+"Z");
  tick();  // Set immediately to avoid blank state
  const id = setInterval(tick, 1000);
  return () => clearInterval(id);
}, []);
```

This creates a ticking clock in the top bar. The effect runs once (`[]` dependency), starts an interval, and cleans it up on unmount or re-render.

### useRef: SVG DOM Access

```javascript
const svgRef = useRef(null);

// Later, in event handler:
const rect = svgRef.current?.getBoundingClientRect();
```

The ref allows direct DOM access for:
- **Hit testing:** Determining if a click is within the interactive zone
- **Coordinate scaling:** Converting client coordinates to SVG viewBox coordinates
- **Performance:** Avoiding synthetic event pooling delays (though modern React has improved this)

### useCallback: Event Handler Memoization

```javascript
const getAngle = useCallback((cx, cy) => {
  const r = svgRef.current?.getBoundingClientRect();
  if (!r) return 0;
  const sx = 400 / r.width, sy = 400 / r.height;
  return toDeg(Math.atan2(
    (cx - r.left) * sx - CX,
    -((cy - r.top) * sy - CY)
  ));
}, []);
```

**Why memoize?** The `getAngle` function is referenced in multiple event handlers (`onCalcDown`, re-defined handlers inside `onDown`). Memoizing ensures:
- The function reference doesn't change between renders (unless dependencies change)
- Event listeners aren't repeatedly de-created and re-attached
- Dependency arrays in child functions remain stable

### Event Handlers: Interaction Logic

#### Calculator Wheel Drag

```javascript
const onCalcDown = useCallback(e => {
  if (side !== "calc") return;
  const c = e.touches?.[0] ?? e;
  const rect = svgRef.current?.getBoundingClientRect();
  if (!rect) return;
  
  const sx = 400 / rect.width, sy = 400 / rect.height;
  const x = (c.clientX - rect.left) * sx - CX;
  const y = (c.clientY - rect.top) * sy - CY;
  const dist = Math.sqrt(x * x + y * y);
  
  if (dist < 62 || dist > 151) return;  // Only in annulus zone
  e.preventDefault();
  
  const sa = getAngle(c.clientX, c.clientY), sr = rot;
  
  const move = ev => {
    const e2 = ev.touches?.[0] ?? ev;
    setRot((((sr + getAngle(e2.clientX, e2.clientY) - sa) % 360) + 360) % 360);
  };
  
  const up = () => {
    window.removeEventListener("mousemove", move);
    window.removeEventListener("mouseup", up);
    window.removeEventListener("touchmove", move);
    window.removeEventListener("touchend", up);
  };
  
  window.addEventListener("mousemove", move);
  window.addEventListener("mouseup", up);
  window.addEventListener("touchmove", move, { passive: false });
  window.addEventListener("touchend", up);
}, [side, rot, getAngle]);
```

**Breaking down this handler:**

1. **Multi-touch support:** `e.touches?.[0] ?? e` works for both touch and mouse events
2. **Hit test:** Only allow drag in the annulus (ring) between radii 62 and 151
3. **Starting angle:** `sa` captures the starting angle relative to circle center
4. **Drag calculation:** As the mouse/touch moves, recompute the angle and update `rot` by the delta
5. **Modulo arithmetic:** The double-mod `((. % 360) + 360) % 360` handles wrap-around correctly (e.g., rotating from 350° past 0° to 10° wraps smoothly)
6. **Event cleanup:** Remove listeners on mouse/touch up to prevent memory leaks

### Wind Grid Interactions

The wind side has two independent interaction zones:

#### Inner Zone: Vertical Offset (Wind Speed Input)

```javascript
if (dist < 112) {
  const sy0 = c.clientY, s0 = windOffset;
  const move = ev => {
    const e2 = ev.touches?.[0] ?? ev;
    setWindOffset(Math.max(-130, Math.min(130, s0 + (e2.clientY - sy0) * sy)));
  };
  // ... attach listeners
}
```

Vertical drag adjusts the grid's vertical offset, simulating the physical E6-B's slide mechanism for setting wind speed.

#### Outer Zone: Azimuth Rotation (Wind Direction Input)

```javascript
} else {
  const sa = getAngle(c.clientX, c.clientY), swa = windAngle;
  const move = ev => {
    const e2 = ev.touches?.[0] ?? ev;
    setWindAngle((((swa + getAngle(e2.clientX, e2.clientY) - sa) % 360) + 360) % 360);
  };
  // ... attach listeners
}
```

Rotational drag adjusts the azimuth plate, reorienting the wind vector direction.

#### Click to Mark Wind Dot

```javascript
const onClick = useCallback(e => {
  const rect = svgRef.current?.getBoundingClientRect();
  if (!rect) return;
  const sx = 400 / rect.width, sy = 400 / rect.height;
  const x = (e.clientX - rect.left) * sx, y = (e.clientY - rect.top) * sy;
  if (Math.sqrt((x - CX) ** 2 + (y - CY) ** 2) < 112) {
    setWindDot({ x, y });
  }
}, [setWindDot]);
```

Single click marks a point on the grid; the distance from center is converted to knots.

---

## Interactive Mechanics

### Touch & Pointer Event Handling

Modern web applications must support multiple input methods:

```javascript
const c = e.touches?.[0] ?? e;  // Touch event or mouse event
```

This pattern:
- Checks for `touches` array (indicating touch event)
- Falls back to the event itself (mouse event with `clientX`, `clientY`)
- Works on desktop (mouse), tablet (touch + mouse), and mobile (touch)

### Coordinate System Transformation

The critical challenge in interactive SVG is **mapping browser coordinates to SVG coordinates**:

```javascript
const rect = svgRef.current?.getBoundingClientRect();
const sx = 400 / rect.width;         // Scale factor: viewBox width / rendered width
const sy = 400 / rect.height;        // Scale factor: viewBox height / rendered height
const x = (c.clientX - rect.left) * sx - CX;  // Client to SVG, then relative to center
const y = (c.clientY - rect.top) * sy - CY;
```

**Example:** If the SVG renders at 300×300px but has viewBox 400×400:
- `sx = 400 / 300 = 1.33`
- A click at 150px from the left (viewport) → 150 × 1.33 = 200 (viewBox) → 200 - 200 = 0° (at center)

### Drag Mechanics

The E6-B uses a **differential drag model**:

1. **Record starting state:** `sa` (starting angle), `sr` (starting rotation)
2. **On each move event:** Calculate new angle and compute delta: `newRot = sr + (newAngle - sa)`
3. **Bound the result:** Apply modulo for wrap-around, max/min constraints for sliders

This approach has advantages:

- **Intuitive:** The instrument rotates *under* the cursor, staying centered
- **Continuous feedback:** Every move event updates the display, creating a fluid experience
- **No state drift:** The current state is always the source of truth; this is more robust than tracking cumulative deltas

### Transient Animation States

The flip animation from calculator to wind side uses a transient state:

```javascript
const flip = () => {
  setTrans(true);  // Trigger CSS scale animation
  setTimeout(() => {
    setSide(s => s === "calc" ? "wind" : "calc");  // Switch side
    setTrans(false);  // End animation
  }, 200);
};
```

The `trans` flag drives a CSS transition:

```javascript
transform: trans ? "scale(0.96)" : "scale(1)",
transition: "transform 0.2s ease",
```

The effect:
1. **Down:** Scale to 96% (briefing effect)
2. **Switch:** Change the rendered content (`side`)
3. **Up:** Return to 100% scale

This creates a quick, visual "flip" transition that masks the content switch.

---

## Responsive Design System

### Media Query Strategy

The application uses a **single breakpoint** at 780px:

```css
@media (min-width: 780px) {
  .e6b-body {
    flex-direction: row;
    align-items: flex-start;
    justify-content: center;
    gap: 20px;
  }
  .e6b-computer-col {
    flex: 0 0 auto;
    width: min(50vw, 480px);
  }
  .e6b-info-col {
    flex: 1 1 0;
    min-width: 260px;
    max-width: 420px;
    display: flex;
    flex-direction: column;
    gap: 10px;
  }
}

@media (max-width: 779px) {
  .e6b-computer-col {
    width: min(92vw, 460px);
  }
  .e6b-info-col {
    width: min(92vw, 460px);
    display: flex;
    flex-direction: column;
    gap: 10px;
  }
}
```

**Desktop (≥780px):** Side-by-side layout with the dial on the left and info panels on the right. The computer is capped at 480px, and the info column at 420px, allowing room for margins on ultra-wide screens.

**Mobile (<780px):** Stacked layout with the dial centered and panels below. Everything scales to 92vw (leaving 4% margin on each side) but capped at 460px to keep text readable.

### Responsive Typography

```javascript
fontSize:"10px"    // Labels on SVG
fontSize:"9px"     // Data row labels
fontSize:"12px"    // Data values
fontSize:"36px"    // Large rate value
```

Text sizes are fixed in px, not responsive, because:
1. The SVG scales infinitely via viewBox
2. Scaling the SVG scales the text proportionally
3. No media query needed; the same `.jsx` renders at all sizes

### Viewport Units & `clamp()`

While not used extensively here, the codebase could benefit from `clamp()` for truly fluid scaling:

```css
width: clamp(260px, 92vw, 480px);  /* min, preferred, max */
```

This ensures:
- Minimum width of 260px (readable on tiny screens)
- Scales with viewport (92vw = 92% of viewport width)
- Capped at 480px (preserves readability on very large screens)

---

## Performance Optimization

### SVG Performance Considerations

#### Geometry Caching

The `MARKS` array is computed once at module level:

```javascript
function mkMarks() { /* ... */ }
const MARKS = mkMarks();  // Computed once, never recomputed
```

This avoids recreating 90 scale mark objects on every render. Rendering the marks still happens on every re-render (since they're in the JSX), but the *computation* is one-time.

#### Path Data Simplification

Rounding to 3 decimal places:

```javascript
return [
  +(CX + r * Math.sin(a)).toFixed(3),
  +(CY - r * Math.cos(a)).toFixed(3)
];
```

This keeps SVG attributes compact and reduces parser overhead:

```svg
<!-- 3 decimals: -->
<line x1="200.000" y1="50.000" x2="200.000" y2="45.000"/>

<!-- vs. full precision (15+ decimals): -->
<line x1="200.0000000000001" y1="50.000000000003" x2="200.0000000000001" y2="45.000000000003"/>
```

#### Filter Optimization

SVG filters (`<feDropShadow>`, `<feGaussianBlur>`) are GPU-accelerated in modern browsers but can be expensive. This app uses only 3 unique filters, applied sparingly:

- `#ds` – drop shadow on main bezel (1 element)
- `#glow` – glow on pointers and major marks (5–10 elements)
- `#greenGlow` – green glow on wind side (5–10 elements)

#### Clip Paths

The wind grid uses a circular clip path to mask the grid lines and wind dot:

```jsx
<clipPath id="gridClip">
  <circle cx={CX} cy={CY} r={111}/>
</clipPath>

<g clipPath="url(#gridClip)">
  {/* Only content inside the circle is rendered */}
</g>
```

This prevents grid lines from spilling outside the circle boundary and reduces rendering area.

### React Performance Considerations

#### useCallback Memoization

```javascript
const getAngle = useCallback(fn, []);  // Never recreated
const onCalcDown = useCallback(fn, [side, rot, getAngle]);  // Recreated only if dependencies change
```

This prevents unnecessary event listener attachment/detachment.

#### Key Props in Lists

```javascript
MARKS.map(({ v, a, ... }) => (
  <g key={v}>  // <- stable key
    {/* ... */}
  </g>
))
```

Using `key={v}` (the mark value) ensures React's reconciliation algorithm doesn't unnecessarily re-mount SVG elements when the list is re-rendered.

#### Functional Sub-components

Functions like `ScaleMarks()` and `Windows()` are pure (no side effects):

```javascript
function ScaleMarks({ outerR, color, accentColor, labelR }) {
  // Renders 90 marks; no hooks, no side effects
  return MARKS.map(/* ... */);
}
```

Rendering is deterministic—same props produce same output—allowing React to optimize if converted to `React.memo()` in the future.

### Bundle Size

The application has **zero external dependencies** beyond React and ReactDOM:

```json
{
  "dependencies": {
    "react": "^19.2.0",
    "react-dom": "^19.2.0"
  }
}
```

The built bundle is approximately:

- React + ReactDOM: ~120 KB (pre-gzip)
- e6b.jsx + styles: ~30 KB
- **Total**: ~150 KB (~40 KB gzip)

This small footprint is suitable for distribution:
- Over slow 3G networks (E6-B is used in remote areas; fast loading is important)
- As an offline-capable app (cache the bundle, run without internet)
- As a white-label tool (embed in pilot training platforms)

---

## Design Tokens & Visual Language

### Color System

```javascript
const VR = {
  accent:  "#4da6ff",  // Primary (blue) – interactive, focused elements
  green:   "#4ade80",  // Secondary (green) – wind side, alternative mode
  amber:   "#fbbf24",  // Tertiary (amber) – warnings, complex directions
  text:    "#e2e8f0",  // Primary text (light gray)
  textSub: "#94a3b8",  // Secondary text (muted gray)
  muted:   "#64748b",  // Tertiary text (very muted)
  divider: "rgba(255,255,255,0.06)",  // Subtle borders
  font:    "'Fira Code', 'JetBrains Mono', monospace",
};
```

**Design rationale:**

- **Dark theme:** Dark backgrounds (#080b0f = almost black) reduce eye strain during night flight
- **High contrast:** Text colors (e2e8f0, 94a3b8) stand out against dark backgrounds
- **Semantic color:** Blue = calculator (primary), Green = wind (alternative)
  - Pilots quickly learn: "Blue side is math, green side is wind"
  - Colorblind-friendly: In addition to color, UI clearly separates modes
- **Monospace font:** "Fira Code" is a professional monospace, implying precision and authority
  - Fallback to "JetBrains Mono" and generic monospace for better compatibility

### Typography

All text uses "Fira Code" monospace:

```css
font-family: 'Fira Code', 'JetBrains Mono', monospace;
```

**Why monospace?**

1. **Fits the theme:** E6-B is a precision instrument; monospace = precision
2. **Readability:** Fixed-width fonts are easier to scan at small sizes (8–12px)
3. **Numbers:** Monospace aligns columns of numbers (rate, time, distance)
4. **Loading:** "Fira Code" is a single font; no multiple weights/styles needed (fast loading)

### Badge Component

```javascript
function Badge({ color, children }) {
  const map = {
    blue:  ["rgba(77,166,255,0.12)","#4da6ff","rgba(77,166,255,0.3)"],
    green: ["rgba(74,222,128,0.1)", "#4ade80","rgba(74,222,128,0.28)"],
    amber: ["rgba(251,191,36,0.1)", "#fbbf24","rgba(251,191,36,0.28)"],
  };
  const [bg,fg,border] = map[color] ?? map.blue;
  return (
    <span style={{
      background: bg,
      color: fg,
      border: `1px solid ${border}`,
      borderRadius: "4px",
      fontSize: "9px",
      fontFamily: VR.font,
      fontWeight: "700",
      padding: "2px 8px",
      letterSpacing: "0.5px",
      WebkitFontSmoothing: "antialiased"
    }}>
      {children}
    </span>
  );
}
```

Semantically labeled badges (`<Badge color="blue">CALC MODE</Badge>`) provide:
- Visual hierarchy (badges stand out from text)
- Color coding (reinforcing mode semantics)
- Compact labeling (8–9px for minimal real estate use)

---

## Accessibility & UX Considerations

### Touch Accessibility

The application prioritizes touch interaction:

```javascript
const c = e.touches?.[0] ?? e;  // Touch events first
window.addEventListener("touchmove", move, { passive: false });  // Enable preventDefault
```

Features:

- **Touch-first event handling:** Touch events take precedence
- **Large touch targets:** The draggable annulus is 89 units wide (≈35% of viewBox)
- **Visual feedback:** Pointers and glows provide immediate feedback
- **No hover states:** Hover is unreliable on touch; all interactivity is press-based

### Keyboard Accessibility

**Current limitation:** The application has no keyboard controls. A future enhancement could add:

```javascript
const onKeyDown = (e) => {
  if (e.key === 'ArrowRight') setRot(rot => (rot + 5) % 360);
  if (e.key === 'ArrowLeft') setRot(rot => (rot - 5) % 360);
  if (e.key === 'ArrowUp') setWindOffset(o => Math.min(130, o + 5));
  if (e.key === 'ArrowDown') setWindOffset(o => Math.max(-130, o - 5));
};
```

### Color Contrast

All text meets WCAG AA standards:

| Element | Foreground | Background | Ratio |
|---------|-----------|-----------|-------|
| Primary text | #e2e8f0 | #080b0f | 17.5:1 ✓ |
| Secondary text | #94a3b8 | #080b0f | 7.2:1 ✓ |
| Accent (blue) | #4da6ff | #080b0f | 5.8:1 ✓ |

Ratios well above 4.5:1 (AA) and many above 7:1 (AAA).

### Reduced Motion

For users with vestibular disorders, flashing/spinning animations can be problematic. A future update should respect `prefers-reduced-motion`:

```javascript
const prefersReducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches;

const flip = () => {
  setTrans(!prefersReducedMotion);  // Skip animation if user prefers
  setTimeout(() => {
    setSide(s => s === "calc" ? "wind" : "calc");
    setTrans(false);
  }, prefersReducedMotion ? 0 : 200);
};
```

### Status Messages

The status bar at the bottom provides context-sensitive hints:

**Calculator side:**
```
↺ DRAG INNER WHEEL  |  TOUCH ENABLED  |  LOG BASE-10
```

**Wind side:**
```
DRAG RING → HDG  |  SLIDE ↕ → WSPD  |  CLICK → MARK
```

These messages guide users unfamiliar with the E6-B, reducing the learning curve.

---

## Future Enhancements & Extensibility

### Refactoring: Component Extraction

The current monolithic 820-line component could be split:

```
src/
├── components/
│   ├── CalcSVG.jsx        # Calculator dial SVG
│   ├── WindSide.jsx       # Wind grid SVG
│   ├── ReadoutPanel.jsx   # Calculation display
│   ├── WindPanel.jsx      # Instructions
│   ├── StatusBar.jsx      # Tips
│   └── Badge.jsx          # Reusable badge
├── hooks/
│   ├── useE6BCalculations.js   # Math logic
│   ├── useTouchDrag.js         # Drag event handling
│   └── useSVGCoordinates.js    # Coordinate transforms
├── utils/
│   ├── math.js            # logA, polar, scale marks
│   ├── design.js          # VR tokens, GLOBAL_STYLE
│   └── constants.js       # CX, CY, marks data
└── E6B.jsx                # Root component
```

Benefits:
- **Testability:** Individual functions are easier to unit test
- **Maintainability:** Smaller files, clearer responsibilities
- **Reusability:** Sub-components usable in other projects
- **Collaboration:** Team members can work on different components in parallel

### Feature: Preset Calculations

Add buttons for common scenarios:

```javascript
const presets = [
  { name: "60 NM in 30 min", rot: logA(120) },
  { name: "Climb to 5000 ft", rot: logA(500) },
  { name: "5 knot headwind", windOffsetperset: -25 },
];
```

Clicking a preset instantly aligns the wheel.

### Feature: Calculation History

Track and display recent calculations:

```javascript
const [history, setHistory] = useState([]);

const logCalculation = useCallback((rate, time, distance) => {
  setHistory(h => [{rate, time, distance, timestamp: Date.now()}, ...h].slice(0, 10));
}, []);
```

### Feature: E6-B Manual Overlay

Include step-by-step guides for common tasks:
- "How to calculate groundspeed"
- "How to determine wind correction angle"
- "How to convert altitude corrected for temperature"

### Feature: Dark/Light Theme Toggle

Currently hardcoded dark theme. A theme toggle would allow:

```javascript
const [theme, setTheme] = useState('dark');

const themeVR = theme === 'dark' ? VR_DARK : VR_LIGHT;
```

### Feature: Unit Preferences

Allow users to toggle between:
- Nautical miles ↔ Statute miles ↔ Kilometers
- Knots ↔ MPH ↔ km/h
- Celsius ↔ Fahrenheit

### Feature: Export & Share

Allow users to screenshot or export their calculations:

```javascript
const exportSVG = () => {
  const svg = svgRef.current;
  const data = new XMLSerializer().serializeToString(svg);
  const blob = new Blob([data], { type: 'image/svg+xml' });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = `e6b-${Date.now()}.svg`;
  a.click();
};
```

### Feature: Mobile App (PWA)

Convert to a Progressive Web App:

```json
{
  "name": "E6-B Flight Computer",
  "short_name": "E6-B",
  "start_url": "/",
  "display": "standalone",
  "icons": [
    { "src": "/icon-192.png", "sizes": "192x192", "type": "image/png" },
    { "src": "/icon-512.png", "sizes": "512x512", "type": "image/png" }
  ],
  "theme_color": "#4da6ff",
  "background_color": "#080b0f"
}
```

Benefits:
- Install on home screen
- Offline functionality (with service worker)
- Native-like experience

---

## Deployment & Distribution

### Build & Optimization

```bash
npm run build
```

Vite produces:

```
dist/
├── index.html           # ~2 KB minified
├── assets/
│   ├── index-xxx.js     # ~40 KB gzip (React + e6b code)
│   └── index-xxx.css    # ~1 KB gzip (minimal overrides)
```

### GitHub Pages Deployment

```bash
# Build
npm run build

# Commit dist/ to gh-pages branch
git subtree push --prefix dist origin gh-pages
```

The app is then live at `https://username.github.io/e6b/`.

### Docker Deployment

For self-hosted or cloud deployment:

```dockerfile
FROM node:18 AS build
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:18
RUN npm install -g serve
COPY --from=build /app/dist /dist
EXPOSE 3000
CMD ["serve", "-s", "/dist", "-l", "3000"]
```

```bash
docker build -t e6b .
docker run -p 3000:3000 e6b
```

### CDN Delivery

Host on a CDN for global availability:

- **Vercel:** Point to GitHub repo, auto-deploy on push
- **Netlify:** Similar setup, with automatic builds
- **AWS CloudFront:** For maximum control and caching

All support HTTPS, automatic gzip, and HTTP/2, making the 40 KB bundle negligible.

---

## Conclusion

The E6-B flight computer demonstrates how React, SVG, and modern CSS can faithfully recreate a beloved analog instrument while enhancing it with digital affordances. The project succeeds through:

1. **Mathematical precision:** Logarithmic scales map correctly using the `logA()` function and polar coordinate transforms.

2. **Visual fidelity:** SVG gradients, filters, and careful color choices recreate the depth and personality of a physical instrument.

3. **Interactive design:** Touch and mouse events are handled seamlessly, with transient animation states and coordinate transforms ensuring fluid operation.

4. **Responsive approach:** A single breakpoint at 780px keeps the UI functional from 280px phones to 4K displays.

5. **Accessible defaults:** Dark theme, high contrast, large touch targets, and descriptive labels make the tool usable by pilots of all experience levels.

6. **Performance-conscious:** Zero external dependencies, efficient SVG primitives, and React hooks keep the bundle small and the frame rate high.

The monolithic architecture works well for this proof-of-concept but would benefit from refactoring as the feature set grows. Additional enhancements—keyboard controls, presets, history, themes, PWA capabilities—are all straightforward to implement on this foundation.

For aviation enthusiasts, software engineers, and educators, the E6-B web app serves as a bridge between analog and digital, preserving the elegance of mechanical computation while leveraging the web platform's reach and ubiquity.

---

## References & Further Reading

- **E6-B History:** [Smithsonian Magazine - The E6-B](https://www.smithsonianmag.com)
- **Aviation Mathematics:** FAA Pilot's Handbook of Aeronautical Knowledge (Chapter 4)
- **React Hooks Documentation:** https://react.dev/reference
- **SVG Specification:** https://www.w3.org/TR/SVG2/
- **Vite Documentation:** https://vitejs.dev/
- **CSS Coordinate Transforms:** MDN Web Docs - `transform`
- **Monospace Fonts:** Fira Code GitHub – https://github.com/tonsky/FiraCode

---

## Appendix: Complete Mathematical Reference

### Scale Mark Generation Algorithm

```javascript
for (let v = 10; v <= 99; v++) {
  const angle = (Math.log10(v) - 1) * 360;
  const isMajor = (v % 10 === 0);
  const isSecondary = (v % 5 === 0);
  
  // Render mark line from (angle, outerR) to (angle, outerR - tickLength)
  // Render label if isMajor or isSecondary, at (angle, labelRadius)
}
```

### Polar to Cartesian Conversion

```javascript
function polarToCartesian(centerX, centerY, radius, angleInDegrees) {
  const angleInRadians = (angleInDegrees - 90) * Math.PI / 180;
  return {
    x: centerX + radius * Math.cos(angleInRadians),
    y: centerY + radius * Math.sin(angleInRadians)
  };
}
```

(Note: The E6B implementation uses a slightly different convention for y-axis orientation.)

### Rate Index Calculation

For a true airspeed value, the rate (distance per hour) is computed as:

```
rate = TAS (knots) = 10^(angle/360 + 1)
```

Where `angle` is the current rotation angle in degrees, ranging from 0–360.

At 60 knots: `(1.778 - 1) × 360 = 280°`

---

##### Contact

**Author:** Kayra Sarı  
**Email:** thekayrasari@gmail.com  
**GitHub:** [@thekayrasari](https://github.com/thekayrasari)

**Repository:** [https://github.com/thekayrasari/e6b](https://github.com/thekayrasari/e6b)
