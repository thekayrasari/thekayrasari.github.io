---
title: "Excalibur WMI Kernel Module: Writing a Real Linux Driver From Scratch"
date: 2025-03-07
description: "How I reverse-engineered my laptop's WMI firmware protocol, discovered an undocumented rainbow mode at 0x7, and rewrote the driver architecture from global variables to a proper per-zone state machine."
tags:
  - linux-kernel
  - device-driver
  - wmi
  - hardware-control
  - kernel-module
  - acpi
  - rgb
  - reverse-engineering
---

[GitHub Repository](https://github.com/thekayrasari/excalibur)

---

## What Changed Since the Last Article

The driver I wrote about before worked. Barely. It had a global variable caching LED state, a single LED classdev for the whole keyboard, a broken brightness mask that corrupted the mode nibble on every write, and a sysfs interface that expected users to pack a raw 64-bit hex integer with the zone, mode, brightness, and RGB all stuffed together with no documentation of the bit layout.

Since then, I did a complete rewrite from scratch — overhauled the architecture, fixed real bugs, and most importantly: reverse-engineered the hardware protocol properly by brute-forcing every possible mode nibble value on my G870. That process uncovered a rainbow mode at `0x7` that was never documented in any upstream Linux patch, any public firmware reference, or even in Mustafa Ekşi's casper-wmi patches that this driver is based on.

This article documents everything: the architecture decisions, the real hardware protocol, the bugs in the original design, and how the hardware discovery actually happened.

---

## The Hardware Interface

Excalibur gaming laptops expose their hardware control through a WMI (Windows Management Instrumentation) block device sitting on top of ACPI. The WMI GUID is:

```
644C5791-B7B0-4123-A90B-E93876E0DAAD
```

All communication goes through a single shared struct:

```c
struct excalibur_wmi_args {
    u16 a0;   /* direction: 0xfa00 READ, 0xfb00 WRITE */
    u16 a1;   /* command:   0x0100 LED, 0x0200 HWINFO, 0x0300 POWERPLAN */
    u32 a2;   /* zone_id (LED) or plan value (POWERPLAN) */
    u32 a3;   /* data word (LED SET) */
    u32 a4;   /* CPU fan RPM (in query response) */
    u32 a5;   /* GPU fan RPM (in query response) */
    u32 a6, rev0, rev1;
};
```

Writes go through `wmidev_block_set()`. Reads go through `wmidev_block_set()` to put the firmware in read mode, then `wmidev_block_query()` to retrieve the response buffer. The kernel allocates the response object as a `union acpi_object`; the driver validates it's an `ACPI_TYPE_BUFFER` of the expected size, copies it out, and frees it.

---

## The LED Data Word — What the Protocol Actually Is

This is where the original article was wrong. The old description said:

```
Bits [31:24]: Zone identifier
Bits [23:16]: Red
Bits [15:8]:  Green
Bits [7:0]:   Blue
```

That's completely incorrect. The zone ID is **not** part of the 32-bit data word at all. It's a separate argument (`a2`) passed alongside the data word (`a3`). The actual 32-bit data word layout is:

```
 31      28 27      24 23    16 15     8 7      0
 ┌─────────┬──────────┬────────┬────────┬────────┐
 │  mode   │  alpha   │  red   │ green  │  blue  │
 │ [31:28] │ [27:24]  │[23:16] │ [15:8] │  [7:0] │
 └─────────┴──────────┴────────┴────────┴────────┘
   nibble    nibble     byte     byte     byte
```

In the driver, these are defined with `GENMASK` and manipulated with `FIELD_PREP` / `FIELD_GET` from `<linux/bitfield.h>`:

```c
#define EXCALIBUR_LED_MODE      GENMASK(31, 28)
#define EXCALIBUR_LED_ALPHA     GENMASK(27, 24)
#define EXCALIBUR_LED_RED       GENMASK(23, 16)
#define EXCALIBUR_LED_GREEN     GENMASK(15, 8)
#define EXCALIBUR_LED_BLUE      GENMASK(7, 0)
```

So to set zone left (`0x03`) to red, full brightness, static mode:

```c
u32 data = FIELD_PREP(EXCALIBUR_LED_MODE,  0x1)  |   /* static */
           FIELD_PREP(EXCALIBUR_LED_ALPHA, 0x2)  |   /* full brightness */
           FIELD_PREP(EXCALIBUR_LED_RED,   0xFF) |
           FIELD_PREP(EXCALIBUR_LED_GREEN, 0x00) |
           FIELD_PREP(EXCALIBUR_LED_BLUE,  0x00);
/* data = 0x12FF0000 */

excalibur_set(drv, EXCALIBUR_SET_LED, 0x03, data);
```

Zone IDs are fixed constants:

| ID | Zone |
|---|---|
| `0x03` | Left keyboard |
| `0x04` | Middle keyboard |
| `0x05` | Right keyboard |
| `0x06` | All keyboard zones (firmware broadcast) |
| `0x07` | Corner LEDs |

Zone `0x06` is the interesting one — writing any LED command to it propagates to all three keyboard zones simultaneously on the firmware side, without needing three separate WMI calls. The driver uses this for brightness changes since the hardware always syncs brightness across all keyboard zones anyway.

---

## Discovering the Rainbow Mode

The mode nibble in bits [31:28] was documented in Mustafa Ekşi's casper-wmi patches up to version 7. The enum stopped at six values:

```c
LED_NORMAL    = 0x10  /* → nibble 1, static */
LED_BLINK     = 0x20  /* → nibble 2 */
LED_FADE      = 0x30  /* → nibble 3, breathing */
LED_HEARTBEAT = 0x40  /* → nibble 4 */
LED_REPEAT    = 0x50  /* → nibble 5, color cycle */
LED_RANDOM    = 0x60  /* → nibble 6, random color */
```

No rainbow. No documentation of anything past `0x6`. I assumed these were correct and implemented the driver accordingly. Everything showed up as static white.

The fix was a `raw` debug sysfs attribute that bypasses all driver parsing and sends an arbitrary 32-bit hex value directly to the hardware:

```c
static ssize_t raw_store(struct device *dev, struct device_attribute *attr,
                         const char *buf, size_t count)
{
    u32 data;
    kstrtou32(skip_spaces(buf), 16, &data);

    mutex_lock(&drv->lock);
    excalibur_set(drv, EXCALIBUR_SET_LED, zone->zone_id, data);
    mutex_unlock(&drv->lock);

    dev_info(dev, "raw: zone=0x%02x data=0x%08x\n", zone->zone_id, data);
    return count;
}
```

Then I ran a brute-force loop across all 16 possible mode nibble values, keeping brightness at `0x2` and color at red `0xFF0000` so any animation was obvious:

```bash
for mode in 0 1 2 3 4 5 6 7 8 9 a b c d e f; do
    echo ${mode}2FF0000 | sudo tee /sys/class/leds/excalibur::kbd_backlight-left/raw > /dev/null
    read -p "Mode 0x$mode — what do you see? " answer
    echo "0x$mode = $answer"
done
```

Results on G870 (BIOS CQ141):

| Nibble | What I saw |
|---|---|
| `0x0` | LEDs off entirely |
| `0x1` | Static red |
| `0x2` | Blinking |
| `0x3` | Breathing |
| `0x4` | Heartbeat double-pulse |
| `0x5` | Color sweeping left to right, jumpy |
| `0x6` | Random color every ~1 second |
| `0x7` | **Rainbow** |
| `0x8–0xf` | Still rainbow (overflow) |

Mode `0x7` was never in any patch, any README, any forum post I could find. The firmware supports it; it was just never tested. The enum in the driver now looks like this:

```c
enum excalibur_led_mode {
    EXCALIBUR_MODE_OFF       = 0,
    EXCALIBUR_MODE_STATIC    = 1,
    EXCALIBUR_MODE_BLINK     = 2,
    EXCALIBUR_MODE_FADE      = 3,
    EXCALIBUR_MODE_HEARTBEAT = 4,
    EXCALIBUR_MODE_WAVE      = 5,
    EXCALIBUR_MODE_RANDOM    = 6,
    EXCALIBUR_MODE_RAINBOW   = 7,
};
```

---

## What Was Wrong With the Original Architecture

### Global state

The original driver kept state in two global variables:

```c
static u32 last_keyboard_led_change;
static u32 last_keyboard_led_zone;
```

`last_keyboard_led_zone` was written but never read anywhere in the code — pure dead state. `last_keyboard_led_change` was a single u32 trying to represent the entire keyboard, which made per-zone color impossible and created race conditions when sysfs attributes were accessed concurrently.

### The brightness mask bug

The brightness setter masked with `0xF0FFFFFF` (clears bits 28–31) while the getter read back with `0x0F000000` (reads bits 24–27). Those are different nibbles. Every brightness write was silently overwriting the mode nibble rather than the alpha nibble. The keyboard would appear to change brightness but the mode value was being corrupted on every write.

### The double-free in `remove`

The original driver called `devm_led_classdev_unregister()` in the `remove` callback on a classdev that was registered with `devm_led_classdev_register()`. The `devm_` prefix means the kernel's managed device resource system already handles cleanup when the device is removed. Calling unregister manually on top of that is a double-free.

### The sysfs interface

The old `led_control` attribute accepted a 64-bit hex integer where the user had to manually pack zone, mode, brightness, and color together in the correct bit positions. There was no way to read the current mode. There was no `DEVICE_ATTR_WO` — it was declared `DEVICE_ATTR_RW` with a `show` function that returned `-EOPNOTSUPP`. The attribute was effectively write-only but pretended to be read-write.

---

## The New Architecture

### State container

All mutable state lives in a heap-allocated struct associated with the WMI device:

```c
struct excalibur_wmi_data {
    struct wmi_device    *wdev;
    bool                  has_raw_fanspeed;
    struct mutex          lock;
    struct excalibur_zone zones[4];
};
```

Retrieved in any callback via `dev_get_drvdata()`. No globals. Concurrent sysfs access is safe because every write path takes the mutex before touching zone state or calling into hardware.

### Per-zone LED devices

Instead of one `led_classdev` for the whole keyboard, the driver registers four separate devices — one per hardware zone:

```
/sys/class/leds/excalibur::kbd_backlight-left/
/sys/class/leds/excalibur::kbd_backlight-middle/
/sys/class/leds/excalibur::kbd_backlight-right/
/sys/class/leds/excalibur::kbd_backlight-corners/
```

Each zone has its own cached state:

```c
struct excalibur_zone {
    struct led_classdev  cdev;
    u8                   zone_id;
    u8                   mode;
    u8                   r, g, b;
};
```

### The commit function

Any change to any field goes through one function that assembles the complete data word from cached state and sends it:

```c
static int excalibur_commit_zone(struct excalibur_wmi_data *drv,
                                  struct excalibur_zone *zone)
{
    u32 data = FIELD_PREP(EXCALIBUR_LED_MODE,  zone->mode)           |
               FIELD_PREP(EXCALIBUR_LED_ALPHA, zone->cdev.brightness) |
               FIELD_PREP(EXCALIBUR_LED_RED,   zone->r)               |
               FIELD_PREP(EXCALIBUR_LED_GREEN, zone->g)               |
               FIELD_PREP(EXCALIBUR_LED_BLUE,  zone->b);

    return excalibur_set(drv, EXCALIBUR_SET_LED, zone->zone_id, data);
}
```

### Keyboard vs corners brightness

The firmware propagates brightness to all three keyboard zones whenever any single keyboard zone is written. The corners zone has fully independent brightness. The driver handles this with two separate `brightness_set` callbacks:

- Keyboard zones use `excalibur_kbd_brightness_set()`, which updates the brightness cache for all three keyboard zones and broadcasts to zone `0x06`.
- Corners use `excalibur_corner_brightness_set()`, which updates only the corners zone and commits directly.

### Sysfs per zone

Each zone now exposes three additional attributes via `.groups` on the `led_classdev`:

```
color           — write-only, 6-digit RRGGBB hex
mode            — read-write, named string
available_modes — read-only, space-separated list
raw             — write-only, 8-digit hex data word, bypasses parsing
```

The mode attribute accepts and returns human-readable names:

```bash
echo rainbow | sudo tee /sys/class/leds/excalibur::kbd_backlight-left/mode
cat /sys/class/leds/excalibur::kbd_backlight-left/mode
# rainbow

cat /sys/class/leds/excalibur::kbd_backlight-left/available_modes
# off static blink fade heartbeat wave random rainbow
```

---

## Fan Speed and the Byte-Swap Problem

Fan speeds are read from `a4` (CPU) and `a5` (GPU) in the `GET_HARDWAREINFO` query response. On newer models the values are native little-endian. On older models with Intel 10th gen or earlier, they come back byte-swapped and need to be reversed:

```c
static u16 excalibur_decode_fanspeed(struct excalibur_wmi_data *drv, u32 raw)
{
    u16 val = (u16)raw;
    if (!drv->has_raw_fanspeed)
        val = (val << 8) | (raw >> 8);
    return val;
}
```

The `has_raw_fanspeed` flag is set per-model via the DMI table. `false` means byte-swap needed (older models), `true` means use raw value (newer models). My G870 with BIOS CQ141 uses `true`.

---

## DMI Matching

The driver uses `dmi_check_system()` during probe to identify the running hardware and set `has_raw_fanspeed` accordingly. The G870 entry was added during this development session — it was not previously in any version of the driver:

```c
{
    .callback    = dmi_matched,
    .ident       = "EXCALIBUR G870",
    .matches     = {
        DMI_MATCH(DMI_SYS_VENDOR,   "EXCALIBUR BILGISAYAR SISTEMLERI"),
        DMI_MATCH(DMI_PRODUCT_NAME, "EXCALIBUR G870"),
        DMI_MATCH(DMI_BIOS_VERSION, "CQ141"),
    },
    .driver_data = (void *)true,
},
```

If your model isn't in the table, the driver emits a warning and defaults to `has_raw_fanspeed = true`. The warning is there specifically to prompt users on older hardware to report their model so it can be added.

---

## The Full Sysfs Interface

### LED (per zone)

| Path | Access | Values |
|---|---|---|
| `.../brightness` | RW | `0`, `1`, `2` |
| `.../color` | WO | `RRGGBB` hex |
| `.../mode` | RW | `off static blink fade heartbeat wave random rainbow` |
| `.../available_modes` | RO | space-separated list |
| `.../raw` | WO | 8-digit hex data word |

### hwmon

| Path | Access | Description |
|---|---|---|
| `hwmon*/fan1_input` | RO | CPU fan RPM |
| `hwmon*/fan2_input` | RO | GPU fan RPM |
| `hwmon*/pwm1` | RW | Power plan: `1` High Power, `2` Gaming, `3` Text Mode, `4` Low Power |

---

## Usage Examples

**Set all zones to rainbow at full brightness:**
```bash
for zone in left middle right corners; do
    echo rainbow | sudo tee /sys/class/leds/excalibur::kbd_backlight-$zone/mode
done
echo 2 | sudo tee /sys/class/leds/excalibur::kbd_backlight-left/brightness
```

**Three-zone RGB split:**
```bash
echo FF0000 | sudo tee /sys/class/leds/excalibur::kbd_backlight-left/color
echo 00FF00 | sudo tee /sys/class/leds/excalibur::kbd_backlight-middle/color
echo 0000FF | sudo tee /sys/class/leds/excalibur::kbd_backlight-right/color
for zone in left middle right; do
    echo static | sudo tee /sys/class/leds/excalibur::kbd_backlight-$zone/mode
done
echo 2 | sudo tee /sys/class/leds/excalibur::kbd_backlight-left/brightness
```

**Gaming profile — max power, red static:**
```bash
echo 1 | sudo tee /sys/class/hwmon/hwmon*/pwm1
for zone in left middle right corners; do
    echo FF0000 | sudo tee /sys/class/leds/excalibur::kbd_backlight-$zone/color
    echo static | sudo tee /sys/class/leds/excalibur::kbd_backlight-$zone/mode
done
echo 2 | sudo tee /sys/class/leds/excalibur::kbd_backlight-left/brightness
```

**Battery profile — low power, dim blue breathing:**
```bash
echo 4 | sudo tee /sys/class/hwmon/hwmon*/pwm1
for zone in left middle right corners; do
    echo 0000FF | sudo tee /sys/class/leds/excalibur::kbd_backlight-$zone/color
    echo fade   | sudo tee /sys/class/leds/excalibur::kbd_backlight-$zone/mode
done
echo 1 | sudo tee /sys/class/leds/excalibur::kbd_backlight-left/brightness
```

**Live monitoring dashboard:**
```bash
watch -n 1 '
printf "Power Plan : "; cat /sys/class/hwmon/hwmon*/pwm1 \
    | sed "s/1/High Power/;s/2/Gaming/;s/3/Text Mode/;s/4/Low Power/"
printf "CPU Fan    : "; cat /sys/class/hwmon/hwmon*/fan1_input; echo " RPM"
printf "GPU Fan    : "; cat /sys/class/hwmon/hwmon*/fan2_input; echo " RPM"
for zone in left middle right corners; do
    printf "%s: "; cat /sys/class/leds/excalibur::kbd_backlight-$zone/mode
done
'
```

---

## Installation

```bash
git clone https://github.com/thekayrasari/excalibur
cd excalibur
make
sudo insmod excalibur.ko        # temporary, unloads on reboot

# or for permanent install:
sudo ./install.sh install
```

Kernel 5.15+ required. `devm_mutex_init()` requires 6.4+.

Verify it loaded:
```bash
lsmod | grep excalibur
dmesg | grep excalibur
ls /sys/class/leds/ | grep excalibur
```

---

## Known Limitations

**No firmware LED read-back.** The firmware has no mechanism to return current LED state. If lighting is changed via a hardware hotkey, the driver's cache becomes stale. Writing any sysfs attribute restores the driver's last known state.

**No per-key RGB.** The hardware exposes three keyboard zones and one corner zone. Per-key control is not supported at the WMI level.

**No direct fan control.** Fan curves are managed entirely by firmware and tied to the active power plan. The driver can select the plan; it cannot set fan speeds or custom curves.

**Mode nibble values may vary by model.** The values above were confirmed by hardware brute-force on the G870 with BIOS CQ141. Older models may use different mappings. If modes behave unexpectedly on your hardware, use the `raw` attribute to probe them manually.

---

## Adding Your Model

If your Excalibur model isn't supported, get your DMI strings:

```bash
sudo dmidecode -s system-product-name
sudo dmidecode -s bios-version
```

Add an entry to `excalibur_dmi_list[]` in `excalibur.c`, build, and open a pull request. If your model's mode nibble values differ from the table above, run the brute-force loop using the `raw` attribute and include the results in the PR.

---

## Acknowledgments

The firmware protocol was first reverse-engineered by Mustafa Ekşi in the casper-wmi project and associated Linux kernel patch series (v1 through v7). This driver extends that work with a corrected bit layout, proper per-zone architecture, mutex protection, and the addition of the previously undocumented rainbow mode at nibble `0x7`, confirmed on G870 hardware.

---

*© 2025 Kayra Sarı — [thekayrasari@gmail.com](mailto:thekayrasari@gmail.com)*