# Standalone BLE Touchpad — ZMK Firmware

Standalone Bluetooth touchpad using a [HolyKeebs TPS43 kit](https://holykeebs.com/products/touchpad-module) and an NRF52840 SuperMini.
Advertises as a separate BLE pointing device — completely independent from any split keyboard (e.g. Aurora Sofle v2).

Uses the [suchobits fork](https://github.com/suchobits/zmk-driver-azoteq-iqs5xx) of the Azoteq IQS5xx ZMK driver, which adds **I2C polling fallback** so no RDY pin is needed.

## Hardware

| Component | Description |
|---|---|
| NRF52840 SuperMini | nice!nano v2-compatible MCU |
| [HolyKeebs TPS43 kit](https://holykeebs.com/products/touchpad-module) | Azoteq IQS5xx touchpad + adapter PCB + FFC cable + 3D-printed mount |
| LiPo battery | Optional — for wireless operation. Can also run via USB-C |

## Wiring

Follow the standard [HolyKeebs touchpad guide](https://docs.holykeebs.com/guides/touchpad-module/) — solder the adapter PCB onto the SuperMini and connect the FFC cable. Only **4 signals** are needed:

```
TPS43 adapter PCB         NRF52840 SuperMini
─────────────────         ──────────────────
VCC  ──────────────────►  VCC  (3.3V)
GND  ──────────────────►  GND
SDA  ──────────────────►  D2   (P0.17)
SCL  ──────────────────►  D3   (P0.20)
```

The adapter PCB routes VCC, GND, SDA and SCL from the FFC cable to the Pro Micro header. The driver polls the sensor over I2C — no extra RDY or RESET wires required.

> **Optional:** If you want interrupt-driven mode (lower latency, lower power), you can wire the RDY pin from the FFC connector to a free GPIO and uncomment `rdy-gpios` in `touchpad.overlay`.

## Building Firmware

### Via GitHub Actions (recommended)

1. Push this repo to GitHub
2. GitHub Actions builds automatically
3. Download the `touchpad-firmware` artifact (`.uf2` file)

### Locally

```bash
west init -l config
west update
west build -s zmk/app -b nice_nano_v2 -- -DSHIELD=touchpad -DZMK_CONFIG="$(pwd)/config"
```

## Flashing

1. Double-click the RESET button on the NRF52840 SuperMini to enter bootloader
2. A USB device `NICENANO` (or `NRF52BOOT`) will appear
3. Copy the `.uf2` file to it

## Usage

The device advertises as **"Touchpad"** via Bluetooth.

1. Power on the device (USB-C or LiPo)
2. Open Bluetooth settings on your computer
3. Pair with "Touchpad"
4. The touchpad now works as a standalone mouse

### Gestures

| Gesture | Function |
|---|---|
| One finger — drag | Mouse movement |
| One finger — tap | Left click |
| Two fingers — tap | Right click |
| Press and hold | Click and drag |
| Two fingers — drag vertically | Vertical scroll |
| Two fingers — drag horizontally | Horizontal scroll |

## Troubleshooting

Uncomment the logging lines in `touchpad.conf`:

```ini
CONFIG_LOG=y
CONFIG_ZMK_LOG_LEVEL_DBG=y
CONFIG_INPUT_LOG_LEVEL_DBG=y
CONFIG_I2C_LOG_LEVEL_DBG=y
```

Rebuild, flash, and connect via USB. Read logs with:

```bash
# Set west runner if needed
west espressif monitor  # or
screen /dev/ttyACM0 115200
```

## Tuning

Edit `touchpad.overlay` to:

- **Swap axes:** Uncomment `switch-xy;` if the touchpad is rotated 90°
- **Invert direction:** Use `flip-x;` / `flip-y;`
- **Enable interrupt mode:** If you wire RDY, uncomment `rdy-gpios` and set correct GPIO

## Credits

- [AYM1607/zmk-driver-azoteq-iqs5xx](https://github.com/AYM1607/zmk-driver-azoteq-iqs5xx) — original ZMK driver
- [suchobits fork](https://github.com/suchobits/zmk-driver-azoteq-iqs5xx) — polling fallback (no RDY pin needed)
- [HolyKeebs](https://holykeebs.com/) — TPS43 touchpad kit and adapter PCB
- [HolyKeebs touchpad guide](https://docs.holykeebs.com/guides/touchpad-module/) — assembly instructions

## File Structure

```
├── .github/workflows/build.yml     # GitHub Actions CI
├── build.yaml                       # ZMK build matrix
├── config/
│   ├── west.yml                     # West manifest (ZMK + azoteq module)
│   └── boards/shields/touchpad/
│       ├── Kconfig.shield           # Shield registration
│       ├── Kconfig.defconfig        # Shield defaults
│       ├── touchpad.overlay         # Device tree (pins, I2C, sensor)
│       ├── touchpad.conf            # Kconfig (drivers, BLE, etc.)
│       └── touchpad.keymap          # Minimal keymap (no keys)
```
