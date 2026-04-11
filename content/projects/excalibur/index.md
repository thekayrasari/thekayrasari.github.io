---
title: "Excalibur WMI Driver"
description: "Linux kernel WMI driver for Casper Excalibur gaming laptops"
date: 2025-08-23
draft: false
tags: ["linux", "kernel", "driver", "wmi", "acpi", "hardware", "c"]
categories: ["projects"]
---

A low-level Linux kernel WMI driver providing comprehensive hardware control for Casper Excalibur gaming laptops, including per-zone RGB keyboard lighting, real-time fan monitoring, and ACPI-based power plan management.

## Overview

**excalibur-wmi** is a kernel driver that bridges the gap between Linux and the Excalibur laptop's proprietary ACPI/WMI hardware interface. Written in C with supporting Python and shell utilities, it enables full software control over keyboard backlighting, system cooling, and power profiles—features typically locked behind Windows-only firmware.

**Status:** Active development | **Kernel Support:** 5.15+ (6.4+ recommended)

---

## Key Features

### Per-Zone RGB Keyboard Control
- Independent color and animation control across 4 zones: left, middle, right, and corner LEDs
- 8 animation modes: off, static, blink, fade, heartbeat, wave, random, rainbow
- 3-level brightness control with hardware-accelerated fade
- Write-only sysfs interface with hardware caching for efficient state management

### Real-Time Fan Monitoring
- CPU and GPU fan speed readout via hwmon interface (`/sys/class/hwmon/`)
- Automatic byte-order detection (big-endian on older models, native on newer)
- RPM displayed in standard Linux fan monitoring tools (`sensors`, `lm-sensors`)

### Power Plan Management
- 4 hardware power profiles: High Power, Gaming, Text Mode, Low Power
- Firmware-tied fan curves automatically adjust to selected profile
- Single sysfs register write for instant profile switching

### RRExtensive Debugging & Extensibility
- Per-zone raw register access for hardware probing and testing
- Complete DMI-based model quirk table supporting 6+ laptop generations
- Structured logging for troubleshooting and kernel module development learning

---

## Hardware Support

| Model | BIOS | CPU Gen | Status |
|-------|------|---------|--------|
| EXCALIBUR G650 | any | 10th |  Supported |
| EXCALIBUR G670 | any | 10th |  Supported |
| EXCALIBUR G750 | any | 10th |  Supported |
| EXCALIBUR G900 | CP131 | 10th |  Supported |
| EXCALIBUR G870 | CQ141 | 12th |  Supported |
| EXCALIBUR G770 | CP221 | 12th |  Supported |

**Not listed?** The driver loads with a fallback quirk. See the README section *Adding New Models* to add your hardware's profile.

---

## Quick Start

### Installation

**Arch / Manjaro / CachyOS:**
```bash
sudo pacman -S linux-headers  # Install appropriate headers for your kernel variant
git clone https://github.com/thekayrasari/excalibur.git
cd excalibur
sudo ./install.sh install
```

**Ubuntu / Debian:**
```bash
sudo apt install build-essential linux-headers-$(uname -r)
git clone https://github.com/thekayrasari/excalibur.git
cd excalibur
sudo ./install.sh install
```

**Clang-built kernels** (CachyOS, some Arch configs):
```bash
# Installer auto-detects and sets CC=clang LLVM=1
sudo ./install.sh install
```

**Test without installation:**
```bash
make
sudo insmod excalibur.ko
# Module unloads on reboot
```

### Basic Usage

**Set keyboard zone to red static:**
```bash
echo FF0000 > /sys/class/leds/excalibur::kbd_backlight-left/color
echo static > /sys/class/leds/excalibur::kbd_backlight-left/mode
echo 2 > /sys/class/leds/excalibur::kbd_backlight-left/brightness
```

**Switch to Gaming power plan:**
```bash
echo 2 > /sys/class/hwmon/hwmon*/pwm1
```

**Monitor fan speeds:**
```bash
watch -n 1 'cat /sys/class/hwmon/hwmon*/fan1_input /sys/class/hwmon/hwmon*/fan2_input'
```

**Launch interactive control panel:**
```bash
pip install textual
sudo python3 control-panel.py
```

---

## Architecture & Design

### State Management (State Pattern)
All mutable driver state is contained in a single `struct excalibur_wmi_data`, allocated once per device and protected by a single mutex. This eliminates global variables and makes the driver safe for concurrent sysfs access—critical for multi-threaded user-space applications.

### Per-Zone LED Architecture
Each zone is a `struct excalibur_zone` embedding a `struct led_classdev`, allowing standard Linux LED class integration with custom sysfs attributes for animation mode, RGB color, and raw hardware register access.

**Commit Path:**
```
sysfs write (color/mode/brightness)
  ↓
Input validation
  ↓
mutex_lock()
  ↓
Update zone cache
  ↓
excalibur_commit_zone()
  ├→ FIELD_PREP() assemble 32-bit data word
  └→ excalibur_set() → wmidev_block_set() → firmware
  ↓
mutex_unlock()
```

### WMI Command Protocol
The driver encapsulates the ACPI/WMI command structure in `struct excalibur_wmi_args`:
- **a0:** Direction (READ `0xfa00` / WRITE `0xfb00`)
- **a1:** Command code (SET_LED `0x0100`, GET_HARDWAREINFO `0x0200`, POWERPLAN `0x0300`)
- **a2–a3:** Command payload (zone_id, data word, plan value)
- **a4–a5:** Response fields (CPU/GPU fan RPM)

### LED Data Word Format
```
Bits [31:28]  Mode      (animation nibble: 0–7)
Bits [27:24]  Brightness (nibble: 0–2)
Bits [23:16]  Red       (byte: 0–255)
Bits [15: 8]  Green     (byte: 0–255)
Bits [ 7: 0]  Blue      (byte: 0–255)
```

All field manipulations use `FIELD_PREP()` macros—no manual bit-shifting.

---

## Sysfs Interface

### LED Devices
```
/sys/class/leds/excalibur::kbd_backlight-{left|middle|right|corners}/
  ├─ brightness      (RW)  — 0–2
  ├─ max_brightness  (RO)  — always 2
  ├─ color           (WO)  — RRGGBB hex (no #)
  ├─ mode            (RW)  — animation mode name
  ├─ available_modes (RO)  — space-separated list
  └─ raw             (WO)  — debug: raw 32-bit hex data
```

### hwmon Device
```
/sys/class/hwmon/hwmon{N}/
  ├─ fan1_input  (RO)  — CPU fan RPM
  ├─ fan1_label  (RO)  — "cpu_fan"
  ├─ fan2_input  (RO)  — GPU fan RPM
  ├─ fan2_label  (RO)  — "gpu_fan"
  └─ pwm1        (RW)  — power plan (1–4)
```

---

## Known Limitations

- **No LED state read-back:** Firmware doesn't support querying current LED state. If changed via hotkeys, cache becomes stale until next write.
- **No per-key RGB:** Hardware exposes 4 zones only; per-key control not supported by WMI interface.
- **No direct fan control:** Fan curves tied to power plan; only plan selection is exposed.
- **Mode values may vary by model:** Animation mode nibbles confirmed on G870; older models may differ.

---

## Project Structure

```
excalibur/
├── Makefile              # Kernel build configuration
├── control-panel.py      # Textual TUI control center
├── install.sh            # Installation/uninstallation script
├── README.md             # Comprehensive documentation
├── CONTRIBUTING.md       # Contribution guidelines
```

---

## Development & Contributing

**Build Requirements:**
- Linux kernel 5.15+ (6.4+ for `devm_mutex_init`)
- GCC or Clang (auto-detected)
- Standard build-essential / linux-headers packages

**Development Workflow:**
1. Fork the repository
2. Make changes to `excalibur.c` or supporting scripts
3. Test locally: `make && sudo insmod excalibur.ko`
4. Verify with `dmesg | grep excalibur`
5. Submit a PR with detailed changelog

**New Hardware Support:**
See *Adding New Models* in the README for DMI matching and quirk configuration.

---

## Learning Resources

This project serves as an excellent reference for:
- **Kernel driver development:** Complete WMI/ACPI integration with LED core and hwmon
- **Embedded systems:** Hardware protocol decoding and bitfield manipulation
- **Linux power management:** Power plan integration and device state caching
- **Concurrent programming:** Mutex protection and safe multi-threaded access patterns

---

## Links

- **Repository:** [github.com/thekayrasari/excalibur](https://github.com/thekayrasari/excalibur)
- **Author:** Kayra Sari
- **WMI GUID:** `644C5791-B7B0-4123-A90B-E93876E0DAAD`

---

## Example Configurations

### Gaming Profile
```bash
#!/bin/bash
echo 1 | tee /sys/class/hwmon/hwmon*/pwm1
for zone in left middle right corners; do
    echo FF0000 | tee /sys/class/leds/excalibur::kbd_backlight-$zone/color
    echo static | tee /sys/class/leds/excalibur::kbd_backlight-$zone/mode
done
echo 2 | tee /sys/class/leds/excalibur::kbd_backlight-left/brightness
```

### Battery Profile
```bash
#!/bin/bash
echo 4 | tee /sys/class/hwmon/hwmon*/pwm1
for zone in left middle right corners; do
    echo 0000FF | tee /sys/class/leds/excalibur::kbd_backlight-$zone/color
    echo fade | tee /sys/class/leds/excalibur::kbd_backlight-$zone/mode
done
echo 1 | tee /sys/class/leds/excalibur::kbd_backlight-left/brightness
```

See the README for more profile scripts (rainbow, RGB split, off).

---

## Acknowledgments

Hardware protocol discovered through firmware reverse-engineering and extensive brute-force testing on the G870 model. Special thanks to the Linux kernel community for excellent WMI, LED core, and hwmon subsystem documentation.