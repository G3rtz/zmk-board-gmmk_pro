# ZMK Board Module: Glorious GMMK Pro (ANSI)

ZMK board definition for the [Glorious GMMK Pro](https://www.pcgamingrace.com/products/glorious-gmmk-pro-75-barebone-black-reservation) keyboard.

## Hardware

| Feature | Details |
|---------|---------|
| MCU | STM32F303CCT6 (ARM Cortex-M4, 72 MHz) |
| Flash | 256 KB |
| RAM | 40 KB |
| Matrix | 11 rows × 8 columns (col2row) |
| Keys | 83 (ANSI layout) |
| Encoder | Alps EC11 (volume knob) |
| LED Driver | 2× AW20216S via Zephyr SPI bitbang bus (module support required) |
| USB VID:PID | 0x320F:0x5044 |
| Bootloader | STM32 DFU |

## Pin Mapping

### Matrix
- **Columns (outputs):** PA0, PA1, PA2, PA3, PA4, PA8, PA9, PA10
- **Rows (inputs):** PB0–PB10

### Encoder
- **A:** PC15
- **B:** PC14

### USB
- **DM:** PA11
- **DP:** PA12

### LED Driver (AW20216S)
- **SCK:** PA5
- **MOSI:** PA6 (matches QMK; this is not STM32F303 hardware SPI1 MOSI)
- **MISO:** PA7
- **CS1:** PB13 (AW20216S #1)
- **CS2:** PB14 (AW20216S #2)
- **EN:** PC13
- The board DTS uses Zephyr `zephyr,spi-bitbang` so PA6 can be driven as the actual AW20216S MOSI line.

## Usage

Add this module to your `config/west.yml`:

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
      submodules:
        - drivers
  self:
    path: config
```

Then in your `build.yaml`:

```yaml
---
include:
  - board: gmmk_pro
```

## Status

- [x] Matrix scanning (83 keys)
- [x] Encoder (Alps EC11)
- [x] USB HID
- [x] N-Key Rollover
- [x] Flash storage (NVS)
- [x] AW20216S device-tree wiring enabled via SPI bitbang
- [ ] RGB underglow effects (depends on zmk-module-aw20216s/ZMK RGB support)
- [ ] Per-key RGB effects (depends on zmk-module-aw20216s/ZMK RGB support)

## License

MIT
