# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

A **ZMK user config repository** for the "Cute Key Pro" — a 3×3 macropad shield running on a Nice Nano v2 (nRF52840). This repo does not contain the full ZMK source; it holds only the shield definition, keymap, config, and CI workflow. The ZMK firmware source is pulled at build time via west into `.zmk/` (gitignored).

## Repository Structure

```
boards/shields/cute_key_pro/   ← Shield definition (overlay, keymap, conf, Kconfig)
config/west.yml                ← West manifest pinning ZMK v0.3
build.yaml                     ← GitHub Actions build matrix
.github/workflows/build.yml   ← CI workflow (delegates to ZMK's reusable workflow)
zephyr/module.yml              ← Tells Zephyr to treat repo root as a board root
```

## Build Commands

**CI builds** are triggered on push/PR via GitHub Actions, which delegates to `zmkfirmware/zmk/.github/workflows/build-user-config.yml@v0.3`. The matrix in [build.yaml](build.yaml) builds `nice_nano_v2` + `cute_key_pro` shield with the `studio-rpc-usb-uart` snippet.

**Local builds** (from the `.zmk/` directory):

```bash
# First time only
cd .zmk
west init -l ../config
west update

# Build
west build -s zmk/app -b nice_nano_v2 -- \
  -DSHIELD=cute_key_pro \
  -DZMK_CONFIG="$(pwd)/../config" \
  -DSNIPPET=studio-rpc-usb-uart
```

Firmware output: `build/zephyr/zmk.uf2` — copy to the Nice Nano's UF2 bootloader drive to flash.

## Hardware Wiring (3×3 Matrix on Nice Nano v2)

- **Diode direction:** row2col (cathode toward row wire)
- **Row pins:** D5/P0.22, D6/P0.24, D7/P1.00
- **Col pins:** D1/P0.06, D0/P0.08, D2/P0.17
- **Features enabled:** BLE, battery reporting, ZMK Studio (live keymap editing over USB)

## Keymap

Single-layer keymap in `boards/shields/cute_key_pro/cute_key_pro.keymap`:

```
┌───┬───┬───┐
│ A │ B │ C │
├───┼───┼───┤
│ D │ E │STU│  ← STU = ZMK Studio unlock
├───┼───┼───┤
│ G │ H │BT │  ← BT = BT_CLR (enters pairing mode)
└───┴───┴───┘
```

## Adding to the Build Matrix

Edit [build.yaml](build.yaml) under `include:`. Each entry needs `board` and `shield`; optionally add `snippet` and `cmake-args`:

```yaml
include:
  - board: nice_nano_v2
    shield: cute_key_pro
    snippet: studio-rpc-usb-uart
```

## ZMK Version

Pinned to `v0.3` in [config/west.yml](config/west.yml). To upgrade, change the `revision` field and re-run `west update`.
