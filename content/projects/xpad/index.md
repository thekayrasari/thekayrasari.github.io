---
title: "xPad — Next-Generation Electronic Flight Bag"
description: "A standalone, browser-based Electronic Flight Bag integrating SimBrief, VATSIM, and multi-provider charting for flight simulation."
tags: ["aviation", "efb", "vatsim", "simbrief", "typescript", "react", "electron"]
---
# xPad — Next-Generation Electronic Flight Bag
xPad is a standalone Electronic Flight Bag (EFB) purpose-built for flight simulation enthusiasts. Unlike panel-integrated EFBs that are tethered to a specific aircraft addon, xPad operates as an independent cockpit companion—accessible from any browser or as a native desktop application. It unifies flight planning, live traffic awareness, dispatch communications, charting, and document management into a single, meticulously designed interface with advanced split-screen multitasking.
## Flight Planning & OFP Integration
### SimBrief Dispatch Pipeline
xPad connects directly to the SimBrief API using a Pilot ID-based authentication flow. Once linked, the system automatically fetches and parses the latest Operational Flight Plan (OFP), extracting:
- Block fuel, trip fuel, contingency, and reserve breakdowns.
- Zero Fuel Weight (ZFW), payload distribution, and performance-limited weights.
- Full route waypoint tables with coordinates, airways, and altitude constraints—rendered as an interactive, scrollable manifest.
- Alternate airports, ETOPS entry/exit points, and runway analysis data.
### Native Flight Planner
Beyond fetching existing plans, xPad includes a full-featured dispatch form that generates flights directly to SimBrief. Pilots have granular control over Aircraft Type, Registration, SELCAL codes, ETOPS parameters, cost index, and routing preferences—eliminating the need to context-switch to a browser for flight setup.
## Live Radar & VATSIM Traffic
The integrated radar module provides real-time visualization of the entire VATSIM network. By continuously polling the VATSIM data feed, xPad renders every online pilot as a heading-rotated aircraft icon on an interactive map layer. Active ATC controllers are overlaid with their sector boundaries and frequency information, giving pilots true-to-life situational awareness of the airspace around them. The rendering pipeline is optimized for sustained 60 FPS updates, even when tracking thousands of simultaneous connections globally.
## Multi-Provider Flight Charts
xPad eliminates the need for standalone charting applications by embedding direct access to multiple chart providers:
- **Navigraph Charts:** Full integration with Navigraph's IFR/VFR chart library.
- **Chartfox:** Community-driven, free charting directly within the EFB.
- **MSFS Flight Planner:** Native support for the built-in MSFS flight planning tools.
Charts are rendered inline within xPad's split-screen layout, allowing pilots to reference SID/STAR plates while simultaneously monitoring their OFP or live radar.
## Dispatch, Weather & AOC
### Real-Time Weather Intelligence
xPad fetches live METAR and TAF reports for departure, arrival, and alternate airports directly from the Aviation Weather Center. Weather data is presented in both raw and decoded formats, enabling rapid assessment of visibility, ceiling, wind conditions, and significant weather phenomena.
### Company Dispatch Simulation
For virtual airline operations, xPad simulates Airline Operations Center (AOC) communications. After an OFP is loaded, the dispatch module generates realistic loadsheet confirmations. Pilots can transmit standardized IATA Delay Codes (AHM730) back through the interactive AOC feed, adding procedural depth to turnaround operations.
## Documents & Note-Taking
### Hybrid ATC Notepad
Recognizing the fast-paced nature of ATC communications on networks like VATSIM and IVAO, xPad includes a hybrid notepad system. It provides structured input fields for Clearance Delivery (SID, Squawk, Transition Level, QNH) and Taxi instructions. For more complex clearances, pilots can switch to an interactive drawing canvas to rapidly sketch vector headings, holding patterns, and frequency assignments using pointer/touch events, backed by a full stroke-undo history.
### PDF Manual Viewer
xPad includes a built-in PDF viewer for uploading and referencing aircraft manuals, airline SOPs, and custom documentation directly within the EFB—keeping all critical references within arm's reach during flight.
## Technical Architecture
xPad is built primarily in **TypeScript** with a **React** front-end, delivering a component-driven UI architecture that supports dynamic split-screen layouts and real-time state management. A **C#** backend handles SimConnect integration for bridging live telemetry data from Microsoft Flight Simulator. The application is packaged for desktop deployment while maintaining full browser compatibility, ensuring flexibility across cockpit setups ranging from single-monitor laptops to multi-display home simulators.
