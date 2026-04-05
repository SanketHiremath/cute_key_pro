# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

This is a **ZMK user config repository** for the "Cute Key Pro" keyboard project. It does not contain the full ZMK source — instead, it contains:
- The board/shield definitions for the CKP hardware (embedded under `.zmk/zmk/`)
- A `config/west.yml` manifest that pulls ZMK v0.3 from GitHub
- A `build.yaml` that drives GitHub Actions matrix builds
- A `boards/shields/` directory for any future custom shields

The actual ZMK firmware source lives under `.zmk/zmk/` (a west workspace clone).

## Keyboard Variants

Three keyboard variants share the same `ckp` hardware platform (nRF52840):

| Board name  | Form factor | Rows × Cols |
|-------------|-------------|-------------|
| `bt60_v2`   | 60%         | 5 × 15      |
| `bt65_v1`   | 65%         | 5 × 16      |
| `bt75_v1`   | 75%         | 6 × 16      |

Board files live in `.zmk/zmk/app/boards/arm/ckp/`.

## Build Commands

Builds are normally triggered by GitHub Actions (see [.github/workflows/build.yml](.github/workflows/build.yml)), which delegates to `zmkfirmware/zmk/.github/workflows/build-user-config.yml@v0.3`. The matrix is defined in [build.yaml](build.yaml).

To build locally using west (run from the `.zmk/` directory after `west init`/`west update`):

```bash
# Initialize west workspace (first time only)
cd .zmk
west init -l ../config
west update

# Build a specific board variant
west build -s zmk/app -b bt60_v2 -- -DZMK_CONFIG="$(pwd)/../config"
west build -s zmk/app -b bt65_v1 -- -DZMK_CONFIG="$(pwd)/../config"
west build -s zmk/app -b bt75_v1 -- -DZMK_CONFIG="$(pwd)/../config"
```

Firmware output is `build/zephyr/zmk.uf2` — copy to the keyboard's UF2 bootloader drive to flash.

## Adding a New Board/Shield to the Build Matrix

Edit [build.yaml](build.yaml) to add entries under `include:`:

```yaml
include:
  - board: bt60_v2
  - board: bt65_v1
  - board: bt75_v1
```

## Hardware Overview

All three variants share `ckp.dtsi` / `ckp-pinctrl.dtsi`:
- **MCU:** Nordic nRF52840 (BLE 5.0 + USB)
- **Key matrix:** col2row, 16 columns (GPIO0 + GPIO1), up to 6 rows
- **Encoders:** 3 × EC11 rotary encoders (GPIO0)
- **RGB underglow:** 12 × WS2812 LEDs via SPI3 (GPIO0 pin 20)
- **Backlight:** PWM on GPIO0 pin 17
- **Battery monitoring:** ADC channel 2, 100k+100k voltage divider
- **External power control:** GPIO0 pin 13
- **Firmware format:** UF2 (for nRF52840 UF2 bootloader)

## Keymap Editing

Keymap files follow standard ZMK `.keymap` syntax and live next to the board DTS files in `.zmk/zmk/app/boards/arm/ckp/`:

- `bt60_v2.keymap` — selectable ANSI/ISO/ALL_1U/HHKB layout via `#define`
- `bt65_v1.keymap`
- `bt75_v1.keymap`

Layer 0 is the default layer; layer 1 (`raise`) is activated by `MO(1)` and includes media, RGB, backlight, and Bluetooth controls.

## ZMK Version

Pinned to `v0.3` in [config/west.yml](config/west.yml). To upgrade, update the `revision` field and re-run `west update`.
