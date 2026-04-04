# Excalibur WMI Kernel Module: Implementation of a Per-Zone RGB and Hardware Control Driver

**Date:** 2025-03-07  
**Description:** Technical documentation of a Linux WMI driver for Excalibur gaming laptops, featuring a reconstructed hardware protocol, per-zone state management, and the implementation of undocumented firmware lighting modes.  
**Tags:** `linux-kernel`, `device-driver`, `wmi`, `acpi`, `reverse-engineering`

[GitHub Repository](https://github.com/thekayrasari/excalibur)

---

## Hardware Interface Specification

Excalibur laptops utilize a Windows Management Instrumentation (WMI) block for hardware abstraction, interfaced via ACPI. The driver interacts with the firmware through the following GUID:

`644C5791-B7B0-4123-A90B-E93876E0DAAD`

The communication protocol relies on a structured argument block passed to the WMI method.

### The WMI Argument Structure
```c
struct excalibur_wmi_args {
    u16 a0;   /* Operation: 0xfa00 (READ), 0xfb00 (WRITE) */
    u16 a1;   /* Subsystem: 0x0100 (LED), 0x0200 (HWINFO), 0x0300 (POWERPLAN) */
    u32 a2;   /* Target ID: Zone_ID for LEDs or Plan_Value for Power */
    u32 a3;   /* Payload: 32-bit LED Data Word */
    u32 a4;   /* CPU telemetry (RPM) */
    u32 a5;   /* GPU telemetry (RPM) */
    u32 a6, rev0, rev1;
};
```

---

## LED Subsystem Architecture

The driver manages the keyboard and chassis lighting by manipulating a 32-bit data word. Through hardware probing, the bit layout was identified as follows:

### 32-Bit LED Data Word Layout
The word is divided into a high-order mode nibble, a brightness (alpha) nibble, and three 8-bit color channels.



| Bits | Field | Description |
|---|---|---|
| `[31:28]` | **Mode** | Lighting effect (Static, Breath, Rainbow, etc.) |
| `[27:24]` | **Alpha** | Brightness levels (0-2) |
| `[23:16]` | **Red** | 8-bit color intensity |
| `[15:8]` | **Green** | 8-bit color intensity |
| `[7:0]` | **Blue** | 8-bit color intensity |

### Enumerated Lighting Modes
Brute-force testing of the mode nibble on G870 (BIOS CQ141) hardware revealed eight functional states, including a previously undocumented rainbow effect at `0x7`.

```c
enum excalibur_led_mode {
    EXCALIBUR_MODE_OFF      = 0,
    EXCALIBUR_MODE_STATIC   = 1,
    EXCALIBUR_MODE_BLINK    = 2,
    EXCALIBUR_MODE_FADE     = 3,
    EXCALIBUR_MODE_HEARTBEAT = 4,
    EXCALIBUR_MODE_WAVE      = 5,
    EXCALIBUR_MODE_RANDOM    = 6,
    EXCALIBUR_MODE_RAINBOW   = 7, /* Undocumented firmware mode */
};
```

---

## Driver Implementation Details

### State Management
The driver utilizes a heap-allocated container pattern to avoid global state, ensuring thread safety via mutex locking during WMI transactions.

```c
struct excalibur_wmi_data {
    struct wmi_device    *wdev;
    struct mutex          lock;
    struct excalibur_zone zones[4];
};
```

### Per-Zone Classdev Registration
Instead of a monolithic keyboard device, the driver registers four distinct `led_classdev` instances. This allows the Linux userspace to address the Left, Middle, Right, and Corner zones independently.

### Atomic Commit Logic
To ensure hardware synchronization, any attribute change (color, mode, or brightness) triggers a re-assembly of the 32-bit data word using `bitfield.h` macros before committing to the WMI block:

```c
static int excalibur_commit_zone(struct excalibur_wmi_data *drv, 
                                 struct excalibur_zone *zone)
{
    u32 data = FIELD_PREP(EXCALIBUR_LED_MODE,  zone->mode)           |
               FIELD_PREP(EXCALIBUR_LED_ALPHA, zone->cdev.brightness) |
               FIELD_PREP(EXCALIBUR_LED_RED,   zone->r)              |
               FIELD_PREP(EXCALIBUR_LED_GREEN, zone->g)              |
               FIELD_PREP(EXCALIBUR_LED_BLUE,  zone->b);

    return excalibur_set(drv, EXCALIBUR_SET_LED, zone->zone_id, data);
}
```

---

## Hardware Telemetry (hwmon)

The driver integrates with the `hwmon` subsystem to expose fan speeds and power plan management.

### Fan Speed Decoding
Telemetry data in the WMI response requires model-specific handling. Older Intel-based models (10th Gen and earlier) return fan RPM in a byte-swapped format, while newer models (11th Gen+) provide native little-endian values. This is handled via DMI-based hardware identification.

### Power Plan Mapping
Power profiles are controlled via the `pwm1` attribute:
- `1`: High Performance
- `2`: Gaming
- `3`: Balanced (Text)
- `4`: Power Saver

---

## Sysfs Interface Reference

### LED Control (Per-Zone)
| Path | Access | Expected Format |
|---|---|---|
| `.../brightness` | RW | `0` (Off), `1` (Dim), `2` (Full) |
| `.../color` | WO | `RRGGBB` (Hexadecimal) |
| `.../mode` | RW | String (e.g., `rainbow`, `static`) |
| `.../raw` | WO | 32-bit direct hex payload |

---

## Conclusion and Future Extensions

This implementation provides a stable, architecturally sound interface for Excalibur laptops on Linux. Future development aims to expand the DMI matching table for broader model support and investigate potential fan curve overrides, though these currently remain under firmware control.

**License:** GPL-2.0  
**Author:** Kayra Sarı  
**Source:** [GitHub Repository](https://github.com/thekayrasari/excalibur)