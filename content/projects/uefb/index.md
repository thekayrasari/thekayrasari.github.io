---
title: "Universal Electronic Flight Bag (UEFB)"
description: "A universal, network-agnostic Electronic Flight Bag integrating SimConnect, SimBrief, and VATSIM for simulation enthusiasts."
tags: ["aviation", "simconnect", "vatsim", "simbrief", "javascript", "react"]
---

# Universal Electronic Flight Bag (UEFB)

The Universal Electronic Flight Bag (UEFB) is an advanced, network-agnostic cockpit companion application tailored for serious flight simulation pilots. Engineered to operate alongside platforms like Microsoft Flight Simulator (MSFS) and X-Plane, UEFB amalgamates real-time telemetry, operational flight planning, global ATC network data, and aeronautical calculations into a single, cohesive interface.

## System Architecture and Integrations

### SimBrief OFP Parsing
UEFB connects directly to the Navigraph SimBrief API. By fetching the latest Operational Flight Plan (OFP), the system parses raw XML/JSON outputs to construct an interactive, dynamic flight manifest. This includes:
- Block fuel, Zero Fuel Weight (ZFW), and Payload calculations.
- Step-climb profiles, ETOPS parameters, and Runway Analysis.
- An interactive, scrollable waypoint table that cross-references live telemetry to track actual vs. planned progress.

### Live VATSIM Telemetry
The map module does not simply display static routes. It continuously polls the VATSIM data feed, parsing thousands of active connections globally. It renders every online pilot as a heading-rotated aircraft silhouette on an OpenStreetMap base layer. By employing efficient geospatial data structures and canvas rendering, UEFB tracks live aircraft states (callsign, altitude, heading, groundspeed) without arbitrary radius limits, maintaining a seamless 60 FPS update rate.

### AOC and Dispatch Simulation
For virtual airline pilots, UEFB simulates Company Dispatch communications. After an OFP is pulled from SimBrief, the dispatch module simulates receiving a final loadsheet. Furthermore, pilots can transmit standardized IATA Delay Codes (AHM730) directly back to the simulated AOC feed, enhancing the realism of turnaround operations.

## Flight Operations Modules

### Interactive Checklists
UEFB features a robust state-machine for normal procedures. It natively includes comprehensive checklists for advanced airliners: A220, A32X family, A330, and the Boeing 737, 747, 757, and 777 series. The checklist state is persistently saved to local storage, allowing pilots to transition seamlessly between phases (e.g., Before Start, After Takeoff, Approach) without data loss during application reloads or simulator crashes.

### Dynamic Notes Canvas
Recognizing the rapid nature of ATC communications (especially on networks like VATSIM or IVAO), UEFB incorporates a hybrid ATC Notepad. It features structured input fields for Clearance Delivery (SID, Squawk, QNH) and Taxi operations. Alternatively, pilots can switch to an interactive Drawing Canvas to quickly scribble complex clearances, vector headings, and frequencies using pointer/touch events, backed by a dynamic stroke-undo history.

## UI/UX Engineering
With the v1.1 update, the entire interface was overhauled using the Catppuccin Macchiato color palette. The UI is built to be legible under varying ambient light conditions, mimicking the strict design constraints of real-world avionics displays. State persistence is managed meticulously, ensuring that active flight plans and dispatch messages remain intact throughout the duration of a long-haul virtual flight.
