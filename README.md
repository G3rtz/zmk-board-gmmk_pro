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
| LED Driver | 2× AW20216S via SPI1 (not yet supported) |
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

### LED Driver (SPI1)
- **SCK:** PA5
- **MOSI:** PA6 (note: QMK calls this MOSI)
- **MISO:** PA7
- **CS1:** PB13 (AW20216S #1)
- **CS2:** PB14 (AW20216S #2)
- **EN:** PC13

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
- [ ] RGB underglow (AW20216S driver needed)
- [ ] Per-key RGB (AW20216S driver needed)

## License

MIT
