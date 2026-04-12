# Standalone BLE Touchpad — ZMK Firmware

Fristående Bluetooth-touchpad med HolyKeebs TPS43 + NRF52840 SuperMini.
Fungerar som en separat BLE-enhet, **inte** kopplad till Aurora Sofle v2.

## Hårdvara

| Komponent | Beskrivning |
|---|---|
| NRF52840 SuperMini | nice!nano v2-kompatibel MCU |
| HolyKeebs TPS43 kit | Azoteq IQS5xx touchpad + adapter PCB |
| LiPo-batteri | Valfritt, för trådlös drift |

## Kopplingsschema

```
TPS43 adapter PCB         NRF52840 SuperMini
─────────────────         ──────────────────
VCC  ──────────────────►  VCC  (3.3V)
GND  ──────────────────►  GND
SDA  ──────────────────►  D2   (P0.17)
SCL  ──────────────────►  D3   (P0.20)
RDY  ──────────────────►  D4   (P0.22)   ← OBS: Krävs!
```

> **Viktigt:** Du behöver **5 kablar**, inte 4! RDY-pinnen (Data Ready) är
> obligatorisk — drivrutinen använder den som interrupt för att veta
> när ny data finns tillgänglig från touchpaden.

### Valfritt: RESET

Om du vill koppla touchpadens RESET-pin, anslut den till **D5 (P0.24)**
och avkommentera `reset-gpios`-raden i `touchpad.overlay`.

### Om du använder adapter-PCB:et

HolyKeebs adapter PCB:t sitter normalt på Pro Micro-headern via
FFC-kabeln. De 4 lödpunkterna på adaptern ger signalerna som mappas
till Pro Micro-pinnarna. Kontrollera vilka pins adaptern router
SDA, SCL och RDY till — de bör matcha D2, D3 och en av de närliggande
pinnarna. Annars kan du löda direkt från FFC-breakout till MCU:n.

## Bygga firmware

### Via GitHub Actions (rekommenderat)

1. Pusha detta repo till GitHub
2. GitHub Actions bygger automatiskt
3. Ladda ner `touchpad-firmware` artifact (`.uf2`-fil)

### Lokalt

```bash
west init -l config
west update
west build -s zmk/app -b nice_nano_v2 -- -DSHIELD=touchpad -DZMK_CONFIG="$(pwd)/config"
```

## Flasha

1. Dubbelklicka RESET-knappen på NRF52840 SuperMini för att gå till bootloader
2. En USB-enhet `NICENANO` (eller `NRF52BOOT`) dyker upp
3. Kopiera `.uf2`-filen dit

## Använda

Enheten annonserar sig som **"Touchpad"** via Bluetooth.

1. Slå på enheten (USB-C eller LiPo)
2. Öppna Bluetooth-inställningar på din dator
3. Para ihop med "Touchpad"
4. Touchpaden fungerar nu som en fristående mus

### Gester

| Gest | Funktion |
|---|---|
| Ett finger — dra | Musrörelse |
| Ett finger — tryck | Vänsterklick |
| Två fingrar — tryck | Högerklick |
| Håll nedtryckt | Klicka-och-dra |
| Två fingrar — dra vertikalt | Scroll vertikalt |
| Två fingrar — dra horisontellt | Scroll horisontellt |

## Felsökning

Avkommentera loggnings-raderna i `touchpad.conf`:

```ini
CONFIG_LOG=y
CONFIG_ZMK_LOG_LEVEL_DBG=y
CONFIG_INPUT_LOG_LEVEL_DBG=y
CONFIG_I2C_LOG_LEVEL_DBG=y
```

Bygg om, flasha, och koppla in via USB. Läs loggar med:

```bash
# Ställ in west runner om det behövs
west espressif monitor  # eller
screen /dev/ttyACM0 115200
```

## Justering

Redigera `touchpad.overlay` för att:

- **Byta axlar:** Avkommentera `switch-xy;` om touchpaden är roterad 90°
- **Invertera riktning:** Använd `flip-x;` / `flip-y;`
- **Ändra RDY-pin:** Byt `rdy-gpios` till rätt GPIO om du kopplar annorlunda

## Filstruktur

```
├── .github/workflows/build.yml     # GitHub Actions CI
├── build.yaml                       # ZMK build-matris
├── config/
│   ├── west.yml                     # West manifest (ZMK + azoteq-modul)
│   └── boards/shields/touchpad/
│       ├── Kconfig.shield           # Shield-registrering
│       ├── Kconfig.defconfig        # Shield-defaults
│       ├── touchpad.overlay         # Device tree (pinnar, I2C, sensor)
│       ├── touchpad.conf            # Kconfig (drivrutiner, BLE, etc.)
│       └── touchpad.keymap          # Minimal keymap (inga tangenter)
```
