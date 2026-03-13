---
title: "The Dualists: Dual-Readout Calorimetry"
date: 2026-03-13
authors:
  - Kayra Sari
  - Arda Genc
  - Arhan Hasan Ünsal
  - Atakan Korkmaz
  - Beren Duygu Yılmaz
  - Doruk Turan
  - Doruk Utku Tarım
  - Duru Sefa
  - Leman Ece Genclesen
  - Neva Yıldızlı
description: "A student-led experiment at CERN's T9 beamline to demonstrate dual-readout calorimetry — simultaneously measuring scintillation and Cherenkov light to correct hadronic energy measurements on an event-by-event basis, directly connected to FCC-ee detector R&D."
tags: ["physics", "CERN", "particle physics", "calorimetry", "detector R&D", "FCC-ee"]
---

## Overview

**The Dualists** is our team's second submission to the [Beam Line for Schools 2026](https://beamlineforschools.cern.ch/) (BL4S) competition at CERN. The experiment tests the **dual-readout calorimetry** principle — a technique at the frontier of particle detector design — using standard equipment already available at CERN's T9 beamline. No custom hardware required.

The experiment is a collaboration of ten students from **TED Ege College, Aydın, Turkey**, the same team behind the [RadiaTED Compton scattering proposal](/projects/radiated-bl4s).

---

## The Problem: Invisible Energy in Hadronic Showers

Measuring the energy of electrons and photons is well-solved in particle physics — electromagnetic calorimeters do this with excellent precision. Hadrons (pions, protons) are a different story.

When a hadron enters a calorimeter, it initiates a **hadronic shower** with two distinct components:

- **Electromagnetic component** — neutral pions produced in the shower decay almost instantly into photon pairs, seeding small EM sub-showers.
- **Hadronic component** — nuclear breakup, slow protons, neutrons, and nuclear de-excitation. These processes deposit energy in ways that don't produce detectable light: the energy is effectively invisible to the detector.

The critical problem is that the relative size of these two components — the **electromagnetic fraction, f_em** — fluctuates randomly from event to event. One pion shower might be 20% EM; the next might be 60%. Because the detector responds differently to each component, this fluctuation directly broadens the measured energy distribution and sets a hard floor on hadronic energy resolution that cannot be overcome by simply adding more material or better electronics.

---

## The Solution: Dual-Readout

The dual-readout technique exploits a key physical difference between the two shower components:

- **All particles** in the EM component are relativistic (electrons, positrons, photons) and produce both **scintillation light** (from ionisation) and **Cherenkov light** (from exceeding the speed of light in the medium).
- **Slow, heavy particles** in the hadronic component — protons, nuclear fragments — produce scintillation light but are **below the Cherenkov threshold**. They contribute to S but not C.

This means the **C/S ratio** is constant for pure EM showers, and lower and variable for hadronic showers depending on f_em. Measuring both signals simultaneously for each event gives two equations:

```
S = E × [f_em + (h/e)_S × (1 − f_em)]
C = E × [f_em + (h/e)_C × (1 − f_em)]
```

Since (h/e)_C ≪ (h/e)_S, the system can be solved event-by-event to extract both the true energy E and f_em. The result is a dramatic improvement in hadronic energy resolution — correcting the dominant source of smearing rather than just averaging over it.

This is the core principle of the **IDEA detector** (Innovative Detector for an Electron-positron Accelerator), the leading calorimeter concept for CERN's proposed **Future Circular Collider (FCC-ee)**. The RD52/DREAM collaboration has been developing it for over a decade. Our experiment tests the same fundamental idea in a form accessible at BL4S.

---

## Experimental Setup

The experiment runs on the **CERN PS T9 mixed secondary beam**, which delivers electrons, pions, and protons at momenta between 0.5 and 10 GeV/c. The detection chain:

**Trigger scintillators (S1, S2)** — Two plastic scintillators bookend the setup. Their coincidence forms the trigger, ensuring only particles that traverse the full calorimeter are recorded.

**Delay Wire Chambers (DWC1, DWC2)** — Measure the transverse beam position with ~200 μm resolution. Used to select events where the beam hits the calorimeter centre and reject edge events with poor shower containment.

**Threshold Cherenkov Counter (XCET)** — A gas-filled Cherenkov detector at adjustable pressure. Different particle species at a given momentum have different velocities, hence different Cherenkov thresholds. By scanning two pressure settings, electrons, pions, and protons can be identified event-by-event. This serves as the ground truth for validating the dual-readout separation.

**Dual-readout calorimeter module** — The core of the experiment. Interleaved layers of:
  - **Lead-glass blocks** (already available at T9): produce Cherenkov light, read out by photomultiplier tubes → signal C
  - **Plastic scintillator tiles** (already available at T9): produce scintillation light, read out by photomultiplier tubes → signal S

Total depth is ~25 cm (≈ 1 nuclear interaction length for pions). Both signals are digitised by ADCs; for each event we record the pair (S, C).

A key strength of this proposal: **all equipment is standard BL4S inventory**. We bring no custom detectors.

---

## Measurement Plan

Data is collected at six beam momenta: 1, 2, 3, 5, 7, and 10 GeV/c, using both beam polarities to access different particle compositions. At each setting, the XCET pressure is tuned for optimal particle identification. Target: at least 50,000 events per configuration, which takes under a minute at full beam rate.

**Analysis pipeline:**

1. **Particle ID** — Tag each event as electron, pion, or proton using XCET and time-of-flight between S1 and S2.
2. **C vs S scatter plot** — Plot Cherenkov signal against scintillation signal for all events at each momentum. Electrons should form a tight, high-C/S cluster; pions and protons should scatter broadly at lower C/S, with the spread encoding f_em fluctuations.
3. **EM fraction extraction** — The pure electron sample (f_em = 1 by definition) calibrates the C/S scale. Each hadron event's C/S ratio then yields its f_em.
4. **Energy correction** — Apply the dual-readout formula to obtain the corrected energy per event. Compare the energy resolution before and after correction.
5. **Energy dependence** — Repeat across all momenta. At higher energies the average EM fraction increases (logarithmically due to multiplicity effects), which should be directly visible as a shift in the pion C/S distribution.

---

## Expected Results

- Clear separation of electron and hadron populations in the C vs S plane across all momenta.
- A constant electron C/S ratio, confirming the EM calibration anchor.
- The pion C/S distribution shifting upward with energy, directly measuring the rise of average f_em.
- Measurable narrowing of the hadron energy distribution after dual-readout correction — resolution approaching the intrinsic stochastic limit rather than the fluctuation-dominated uncorrected value.
- A distinct C/S signature for protons versus pions, reflecting their different shower development.

Simulated C vs S scatter plots at 2, 5, and 10 GeV/c confirm these expectations are testable within the available statistics.

---

## Scientific Significance

To our knowledge, no previous BL4S experiment has attempted a dual-readout measurement. The combination of lead-glass (Cherenkov) and plastic scintillator (scintillation) available at T9 is a unique opportunity to demonstrate this principle with real beam data at CERN, contributing directly — if in miniature — to the R&D programme for future collider detectors targeting sub-percent hadronic energy resolution.

---

## Team

The Dualists — TED Ege College, Aydın, Turkey  
Arda Genc · Arhan Hasan Ünsal · Atakan Korkmaz · Beren Duygu Yılmaz · Doruk Turan · Doruk Utku Tarım · Duru Sefa · Kayra Sari · Leman Ece Genclesen · Neva Yıldızlı