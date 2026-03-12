# ZMK Board Module: Glorious GMMK Pro (ANSI)

ZMK board definition for the [Glorious GMMK Pro](https://www.pcgamingrace.com/products/glorious-gmmk-pro-75-barebone-black-reservation) keyboard.

## Hardware Overview

| Feature | Details |
|---|---|
| MCU | STM32F303CCT6 (ARM Cortex-M4 @ 72 MHz) |
| Flash | 256 KB |
| RAM | 40 KB |
| Matrix | 11 rows × 8 columns (`col2row`) |
| Keys | 83 (ANSI layout) |
| Encoder | Alps EC11 (volume knob) |
| LED Driver | 2× AW20216S via SPI1 (module support required) |
| USB VID:PID | `0x320F:0x5044` |
| Bootloader | STM32 DFU |

## Pin Mapping

### Matrix
- **Columns (outputs):** `PA0`, `PA1`, `PA2`, `PA3`, `PA4`, `PA8`, `PA9`, `PA10`
- **Rows (inputs):** `PB0`–`PB10`

### Encoder
- **A:** `PC15`
- **B:** `PC14`

### USB
- **DM:** `PA11`
- **DP:** `PA12`

### LED Driver (SPI1)
- **SCK:** `PA5`
- **MOSI:** `PA6`
- **MISO:** `PA7`
- **CS1:** `PB13` (AW20216S #1)
- **CS2:** `PB14` (AW20216S #2)
- **EN:** `PC13`

## Usage

For a standard ZMK user-config workflow, **`config/west.yml` is the canonical manifest path**.

Add this board module there:

```yaml
manifest:
  remotes:
    - name: zmkfirmware
      url-base: https://github.com/zmkfirmware
    - name: g3rtz
      url-base: https://github.com/G3rtz
  projects:
    - name: zmk
      remote: zmkfirmware
      revision: main
      import: app/west.yml
    - name: zmk-board-gmmk-pro
      remote: g3rtz
      revision: main
    - name: zmk-module-aw20216s
      remote: g3rtz
      revision: main
  self:
    path: config
```

Then in your `build.yaml`:

```yaml
---
include:
  - board: gmmk_pro/stm32f303xc/zmk
```

## Enabling RGB / AW20216S

1. Ensure `zmk-module-aw20216s` is present in `config/west.yml`.
2. Refresh dependencies:

   ```sh
   west update
   ```

3. Enable required options in your user config (`config/<shield>.conf` or `config/board.conf`) and add overlays enabling AW20216S nodes as needed.
4. Build again (`west build ...` or GitHub Actions build).

Expected with the AW20216S module: RGB underglow and (eventually) per-key RGB, depending on current ZMK driver support.

## Build Troubleshooting

### 1) `west: command not found`
Install West in your Python environment:

```sh
python3 -m pip install --user west
```

### 2) `error: BOARD ... is not marked as ZMK compatible`
This board includes `CONFIG_ZMK_BOARD_COMPAT=y` in both board targets and ZMK variant defconfig. If you still hit this:
- confirm your manifest pulled the latest board revision,
- run `west update`,
- remove stale build directories before rebuilding.

### 3) AW20216S / RGB not compiling or missing
This is a known ecosystem pain point while support evolves. Ensure:
- `zmk-module-aw20216s` is present in your manifest,
- overlays actually enable SPI/AW20216S nodes,
- your module and ZMK revisions are mutually compatible.

## Status

- [x] Matrix scanning (83 keys)
- [x] Encoder (Alps EC11)
- [x] USB HID
- [x] N-key rollover
- [x] Flash storage (NVS)
- [ ] RGB underglow (AW20216S integration dependent)
- [ ] Per-key RGB (AW20216S integration dependent)

## License

MIT
