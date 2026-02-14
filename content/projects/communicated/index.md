---
title: "Communication On A Vertical Axis Using Cosmic Ray Muons"
date: 2025-03-19
tags: ["particle physics", "muon detection", "wireless communication", "cosmic rays", "Helmholtz coils"]
author: ["Kayra Sari", "Arhan Hasan Ünsal", "Doruk Turan", "Doruk Utku Tarım", "Eren Demirel", "Tuna Şahin", "Uygar Yıldızcı", "Yağmur Sude Mersin", "Kaan Emre Karabacak"]
description: "This paper proposes a novel telecommunication system that uses cosmic ray muons for wireless data transmission through solid obstructions. Presented at CERN Beamline for Schools Competition, 2024."
summary: "We demonstrate a proof-of-concept muon-based wireless communication system that modulates natural atmospheric muon flux using magnetic fields to encode binary information, enabling transmission through solid rock and water."
cover:
    alt: "Muon Communication Experimental Setup"
    relative: true
editPost:
    URL: ""
    Text: "CERN BL4S Competition"
---
---

##### Download

[Download PDF](paper1.pdf)

---

##### Abstract

Communication through dense materials remains a fundamental challenge in fields ranging from underground mining to submarine operations. This paper presents a muon-based wireless communication system that exploits the unique penetrating properties of cosmic ray muons. Using Helmholtz coils to generate controlled magnetic fields (0.12 T), we deflect muon trajectories to encode binary information. A dual scintillator detector system reads out the modulated muon flux, creating a binary communication channel. Our experimental setup, tested at CERN with a 4 GeV/c muon beam, achieves a measurable deflection of 1.1 cm over a 4-meter vertical path, sufficient for reliable signal discrimination. The system addresses hadron contamination through a Cherenkov veto trigger, ensuring clean muon identification. Thermal management calculations confirm operational stability with a temperature rise of 4.67°C/sec during field activation periods. This proof-of-concept demonstrates feasibility for communication in environments where traditional electromagnetic signals fail.

---

##### Figure 2: Experimental Setup Schematic

<img width="1380" height="457" alt="{72C5C886-BC41-43CF-A4FB-FFC0DE95BFED}" src="https://github.com/user-attachments/assets/243da3fc-0054-4199-9631-8be5fff21c5e" />

*Schematic layout showing the vertical muon beam path through Helmholtz coils and dual scintillator detector system for binary readout.*

---

##### Figure 3: Muon Trajectory Deflection

<img width="547" height="681" alt="{C11D4254-A2FB-4FCD-AC58-4D3289A242B6}" src="https://github.com/user-attachments/assets/6193308e-7aaa-4c99-981a-a5d4943325b1" />

*Illustration of muon trajectories before entering, inside, and after exiting the magnetic field region, demonstrating the 1.106 cm deflection over 4 meters.*

---

##### Highlights

+ **Novel Communication Method**: First demonstration of using modulated cosmic ray muons for wireless data transmission
+ **High Penetration**: Exploits muon's ability to penetrate solid obstructions (rock, water) where RF signals fail
+ **Binary Encoding**: ON/OFF magnetic field states create reliable digital readout via scintillator detection
+ **Hadron Rejection**: Cherenkov veto system ensures clean muon identification despite beam contamination
+ **Scalable Design**: Python implementation allows parameter optimization for various real-world scenarios
+ **Thermal Stability**: Calculated 4.67°C/sec rise permits pulsed operation with optional air cooling

---

##### Key Parameters

**Muon Properties:**
- Momentum: 4 GeV/c (±15% spread)
- Lifetime: ~2.2 microseconds
- Mass: ~200× electron mass
- Natural flux: Most abundant charged particles at sea level

**Magnetic System:**
- Configuration: Helmholtz coil pair
- Coil radius: 15 cm
- Number of turns: 1000 per coil
- Current: 20 A
- Magnetic field: 0.119890 T

**Detection System:**
- Deflection distance: 1.106 cm over 4 m
- Cyclotron radius: 111.44 m
- Detector type: Dual scintillators (±muons)
- Veto system: Threshold Cherenkov detector

---

##### Applications

This muon-based communication system enables wireless transmission in challenging environments:

- **Underground Operations**: Mining, tunneling, cave rescue where RF cannot penetrate
- **Submarine Communication**: Deep-sea operations requiring surface contact through water
- **Disaster Scenarios**: Communication through collapsed structures, debris fields
- **Secure Channels**: Natural cosmic ray flux provides inherent signal obscuration
- **Scientific Installations**: Deep underground laboratories, neutrino detectors

---

##### Citation

Sarı, Kayra, Arhan Hasan Ünsal, Doruk Turan, Doruk Utku Tarım, Eren Demirel, Tuna Şahin, Uygar Yıldızcı, Yağmur Sude Mersin, and Kaan Emre Karabacak. 2024. "Communication On A Vertical Axis Using Cosmic Ray Muons." *CERN Beamline for Schools Competition*.

```BibTeX
@article{SUTDDSYMK24,
    author = {Kayra Sarı and Arhan Hasan Ünsal and Doruk Turan and 
              Doruk Utku Tarım and Eren Demirel and Tuna Şahin and 
              Uygar Yıldızcı and Yağmur Sude Mersin and 
              Kaan Emre Karabacak},
    year = {2024},
    title = {Communication On A Vertical Axis Using Cosmic Ray Muons},
    journal = {CERN Beamline for Schools Competition},
    note = {CommunicaTED Team Project}
}
```

---


##### Experimental Methodology

**Magnetic Field Generation:**

The Helmholtz coil produces a uniform magnetic field given by:

B = (0.899176 × 10⁻⁶ × N × I) / r

where N = 1000 turns, I = 20 A, r = 0.15 m, yielding B = 0.119890 T.

**Deflection Calculation:**

The cyclotron radius R = p/(qB) = 111.44 m for 4 GeV/c muons. Using geometric analysis of the circular trajectory through the 30 cm coil region, followed by 4 m straight-line propagation, we calculate a lateral deflection of 1.106 cm.

**Binary Encoding:**

- **Logic '1'**: Magnetic field ON → muons deflected → scintillator signal
- **Logic '0'**: Magnetic field OFF → muons pass undetected → no signal

Extended protocols (e.g., Morse code) use variable pulse durations.

**Particle Identification:**

Threshold Cherenkov detector vetoes hadronic contamination (pions, kaons) in the CERN beam, ensuring only muon events trigger data acquisition.

---

##### Python Implementation

```python
import math

def getDrift():
    """
    Calculate muon deflection for given experimental parameters.
    
    Returns:
        float: Lateral drift in centimeters
    """
    d = 4          # Vertical length of muon trajectory (m)
    I = 20         # Current through coils (A)
    r = 0.15       # Coil radius (m)
    N = 1000       # Number of turns per coil
    k = 8.99176e-7 # Helmholtz constant
    
    T = (I * N * k) / r  # Magnetic field (T)
    p = 4                 # Momentum (GeV/c)
    
    # Cyclotron motion radius
    R = (p * 5.344285) / (1.6 * T)
    
    # Geometric calculation
    A = math.sqrt(R * R - 4 * r * r)
    X = d * 2 * r / A
    
    return 100 * X  # Convert to cm
```

---

##### Acknowledgments

We extend our deepest gratitude to **Bora Isildak** and **Eda Erdogan** for their invaluable technical guidance throughout the development of this experiment. Special thanks to our physics teacher, **Burcu Karabacak**, for introducing us to the CERN Beamline for Schools competition and providing unwavering support.

We sincerely thank **CERN** for making our dream possible by providing access to world-class experimental facilities and for fostering scientific curiosity among high school students worldwide.

---

##### About CommunicaTED

We are a team of senior high school students from Turkey united by our passion for physics and innovation. Motivated by curiosity and a commitment to addressing real-world challenges, we aim to leverage fundamental physics principles—particularly in particle physics—to develop novel solutions across diverse fields. This project represents our collective effort to push the boundaries of wireless communication technology.

**Team Members:** Kayra Sarı, Arhan Hasan Ünsal, Doruk Turan, Doruk Utku Tarım, Eren Demirel, Tuna Şahin, Uygar Yıldızcı, Yağmur Sude Mersin, Kaan Emre Karabacak
