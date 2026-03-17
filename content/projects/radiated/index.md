---
title: "RadiaTED: Compton Scattering & Timing Resolution"
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
description: "A student-led experiment at CERN's T9 beamline to separate geometric and detector-induced contributions to timing resolution in Compton-based TOF systems, with direct implications for next-generation TOF-PET scanners."
summary: "The experiment investigates how the geometry of Compton scattering contributes to the timing resolution of time-of-flight (TOF) detection systems — a question with direct consequences for the precision of PET scanners used in cancer imaging."
tags: ["physics", "CERN", "particle physics", "PET imaging", "Geant4", "radiotherapy"]
---


The experiment investigates how the geometry of Compton scattering contributes to the timing resolution of time-of-flight (TOF) detection systems — a question with direct consequences for the precision of PET scanners used in cancer imaging.

The project is a collaboration of ten students from **TED Ege College, Aydın, Turkey**.

---

## The Problem

In time-of-flight PET (TOF-PET), the location of an annihilation event is inferred from the time difference between two gamma ray detections. Better timing resolution means more precise tumour localisation and a higher signal-to-noise ratio — ultimately allowing more targeted radiotherapy with less damage to healthy tissue.

Timing resolution degrades from two independent sources:

- **Intrinsic detector jitter (σ_det):** electronic noise and the finite response time of the scintillators themselves.
- **Geometric path-length variation (σ_geo):** when Compton-scattered particles travel different distances to the detector depending on their scattering angle, the spread of arrival times broadens the measured distribution even with a perfect detector.

These two contributions add in quadrature: **σ²_total = σ²_geo + σ²_det**. No prior experiment has directly separated them using simultaneous track reconstruction and timestamping within the same event. That is precisely what we set out to do.

---

## Experimental Setup

The experiment is staged at CERN's **T9 beamline** and uses the following detection chain:

1. **Electron Beam** — A GeV-scale electron beam provides the primary particle source. The controllable beam energy reduces systematic uncertainty in the photon spectrum.
2. **Cherenkov Detector** — Tags each incoming electron and provides an independent timing cross-check, without entering the coincidence trigger.
3. **Bremsstrahlung Target** — A **2.5 X₀ tungsten** target (≈ 8.75 mm) converts electrons into a collimated photon flux via Bremsstrahlung. Tungsten's high atomic number (Z = 74) maximises photon yield; 2.5 X₀ is the optimal thickness balancing yield against self-absorption.
4. **Collimator** — Restricts the photon beam to a narrow pencil, defining a precise interaction region and reducing path-length uncertainty.
5. **Tissue-Equivalent Phantom** — A 3 × 3 × 3 cm³ cube of plastic approximating soft tissue (≈10.5% H, 25.6% C, 60.2% O, 3.7% N; density 1.04 g/cm³). Compton scattering is the dominant photon interaction in this material at the relevant energies.
6. **Scintillator t₁** — A 5 mm plastic scintillator tile records the passage of the Compton-scattered electron and produces the first timestamp. Plastic is chosen for its ~2–3 ns fluorescence decay time, far shorter than alternatives like LYSO (40 ns).
7. **Two Delay Wire Chambers (DWC)** — Gas-filled position-sensitive detectors with ~200 μm spatial resolution. They reconstruct the 3D trajectory of the scattered electron, giving the scattering angle θ and the geometric path length L to the second scintillator.
8. **Scintillator t₂** — Records the electron's arrival and produces the second timestamp. The time-of-flight is **Δt = t₂ − t₁**.

---

## Why an Accelerator?

At PET-relevant energies (511 keV), the maximum Compton electron kinetic energy is ~340 keV — corresponding to a range of roughly 1 mm in plastic. Such an electron would be absorbed by the first scintillator and never reach the tracking chambers.

MeV-scale Bremsstrahlung photons from GeV electrons produce Compton recoil electrons with *tens of MeV* of kinetic energy. These are relativistic, minimum-ionising particles that traverse the entire detection chain cleanly. The accelerator also provides the instantaneous photon flux needed to accumulate 10³–10⁴ coincidence events within the allocated beam time — statistically impractical with any laboratory or radioactive source.

The methodology is energy-independent. The σ_geo / σ_det decomposition technique developed here transfers directly to the 511 keV PET regime.

---

## Analysis Method

For each triggered event we record Δt, the DWC hit positions, the scattering angle θ, and the energy deposited in each scintillator.

The geometric path length is computed event-by-event as:

**L = Δz / cos(θ)**

where Δz is the known longitudinal distance between the phantom and Scintillator t₂.

- The standard deviation of the L distribution directly gives **σ_geo ≈ σ(L)/c**.
- Events are then binned by θ. Within a narrow angular bin, path-length variation is suppressed, so the FWHM of the Δt distribution within that bin converges to **σ_det**.
- **σ_geo** is extracted by subtracting σ_det in quadrature from the full-sample FWHM.

The entire decomposition is derived from experimental data, with no simulation input at the analysis stage.

---

## Geant4 Simulation Results

We ran a Geant4 Monte Carlo simulation of the full beamline prior to submission to validate the design and generate falsifiable predictions.

Key results from the simulation (500 MeV electron beam on the tungsten target):

- Compton scattering angle distribution is **strongly forward-peaked**: 81% of events scatter within 10° of the beam axis, with a mean angle of 6.9° (σ = 4.4°). This is consistent with Klein–Nishina predictions at high photon energies and confirms that the DWC and Scintillator t₂ will intercept the vast majority of scattered particles.
- The Bremsstrahlung photon spectrum spans **0.4–97.7 MeV** with a median of 3.8 MeV.
- Energy deposition in the phantom is exclusively from electrons — confirming Compton scattering as the dominant process, with negligible photoelectric or pair-production contamination.
- Using the simulated transverse hit spread (radial σ ≈ 24.9 mm) and the phantom-to-scintillator separation (Δz ≈ 232 mm), the predicted **σ_geo ≈ 83 ps** — roughly 28–41% of the total timing resolution budget for plastic scintillators. This is large enough to be resolved by the coincidence setup and provides the quantitative motivation for the experiment.

The simulation code and output data are publicly available: [github.com/thekayrasari/bl4s-geant](https://github.com/thekayrasari/bl4s-geant).

---

## Expected Outcome

If the statistical target of ~1500 coincidence events is met, this experiment will yield the **first directly measured, track-assisted decomposition of geometric and detector-induced timing contributions** in a Compton-based TOF geometry. The results are intended as experimentally grounded reference data for the optimisation of next-generation TOF-PET systems targeting sub-100 ps total timing resolution.

---

## Team

RadiaTED — TED Ege College, Aydın, Turkey  
Arda Genc · Arhan Hasan Ünsal · Atakan Korkmaz · Beren Duygu Yılmaz · Doruk Turan · Doruk Utku Tarım · Duru Sefa · Kayra Sari · Leman Ece Genclesen · Neva Yıldızlı

*Advisors: Eda Erdoğan, Burcu Karabacak, Güneş Topal, Bora Işıldak*