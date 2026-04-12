# Standalone BLE Touchpad — ZMK Firmware

Standalone Bluetooth touchpad using HolyKeebs TPS43 + NRF52840 SuperMini.
Works as a separate BLE device, **not** connected to Aurora Sofle v2.

## Hardware

| Component | Description |
|---|---|
| NRF52840 SuperMini | nice!nano v2-compatible MCU |
| HolyKeebs TPS43 kit | Azoteq IQS5xx touchpad + adapter PCB |
| LiPo battery | Optional, for wireless operation |

## Wiring Diagram

```
TPS43 adapter PCB         NRF52840 SuperMini
─────────────────         ──────────────────
VCC  ──────────────────►  VCC  (3.3V)
GND  ──────────────────►  GND
SDA  ──────────────────►  D2   (P0.17)
SCL  ──────────────────►  D3   (P0.20)
RDY  ──────────────────►  D4   (P0.22)   ← NOTE: Required!
```

> **Important:** You need **5 wires**, not 4! The RDY pin (Data Ready) is
> mandatory — the driver uses it as an interrupt to know
> when new data is available from the touchpad.

### Optional: RESET

If you want to connect the touchpad's RESET pin, wire it to **D5 (P0.24)**
and uncomment the `reset-gpios` line in `touchpad.overlay`.

### Using the Adapter PCB

The HolyKeebs adapter PCB normally sits on the Pro Micro header via
the FFC cable. The 4 solder points on the adapter provide the signals
that map to the Pro Micro pins. Verify which pins the adapter routes
SDA, SCL, and RDY to — they should match D2, D3, and one of the
adjacent pins. Otherwise, you can solder directly from the FFC breakout to the MCU.

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
- **Change RDY pin:** Change `rdy-gpios` to the correct GPIO if wired differently

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
