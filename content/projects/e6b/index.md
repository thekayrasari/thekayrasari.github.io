---
title: "E6B Flight Computer & Vector Physics Simulator"
description: "An interactive, browser-based E6-B flight computer and vector physics simulator."
tags: ["aviation", "e6b", "physics", "javascript", "matter.js", "vector-math"]
---

# E6B Flight Computer & Vector Physics Simulator

The E6B Flight Computer project is a modernization of the classic analog "whiz wheel" used by aviators since the 1930s. This interactive, browser-based tool translates mechanical slide-rule computations into dynamic JavaScript algorithms, coupled with a real-time vector physics simulator driven by Matter.js. It serves as both a practical flight planning utility and an educational visualizer for the physics of flight.

## Core Computational Modules

### Wind Triangle Kinematics
The fundamental problem the E6B solves is the calculation of heading and ground speed given a desired true course, true airspeed (TAS), and forecast wind conditions. 

The software mathematically models this using 2D vector addition:
$$ \vec{V}_{gs} = \vec{V}_{tas} + \vec{V}_{wind} $$

By breaking the vectors down into their orthogonal components (North/East), the system computes the necessary Wind Correction Angle (WCA). The equations implemented in the solver exactly calculate the required crab angle to maintain the desired track:
$$ WCA = \arcsin\left(\frac{V_{wind} \cdot \sin(Wind_{dir} - Course)}{V_{tas}}\right) $$
Simultaneously, the system continuously outputs precise headwind and crosswind components, which are critical for takeoff and landing performance calculations.

### Atmospheric Modeling
Aircraft performance is highly dependent on air density. The E6B simulator includes rigorous atmospheric models to calculate:
- **Pressure Altitude (PA):** Correcting indicated altitude for non-standard altimeter settings using the standard lapse rate.
- **Density Altitude (DA):** Computing the theoretical altitude at which the air density matches the current non-standard temperature, crucial for engine and aerodynamic performance mapping.
- **True Airspeed (TAS):** Iteratively calculating TAS from Indicated Airspeed (IAS) by factoring in the specific gas constant for dry air and the computed density altitude.

### Time, Speed, Distance, and Fuel
Using foundational kinematic equations, the app computes Estimated Time Enroute (ETE) based on the calculated ground speed and route distance. Furthermore, by linking specific fuel consumption rates, it provides real-time updates on required block fuel and flight duration. 

## Technical Implementation

### Vector Visualization Engine
To bridge the gap between abstract numbers and physical intuition, the application integrates **Matter.js**, a 2D physics engine. The canvas provides a HUD-style instrument cluster overlaying a live vector representation of the wind triangle. As the user manipulates sliders for wind velocity or desired course, the $\vec{V}_{tas}$ and $\vec{V}_{wind}$ vectors are continuously re-rendered and updated at 60 FPS, providing instant visual feedback on how crosswinds deflect the aircraft's trajectory.

### Architecture
The project is built with vanilla JavaScript, emphasizing a zero-dependency architecture. The logic is entirely executed client-side, making it inherently capable of functioning offline—a critical requirement for actual in-flight use. For development, a lightweight Node.js server handles asset serving, ensuring rapid iteration without the bloat of massive bundlers or frameworks. The UI state is bound directly to the computational engine, ensuring that any DOM input immediately cascades through the kinematic solvers and triggers a canvas repaint.
