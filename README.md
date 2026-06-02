# ZMK Board Module: Glorious GMMK Pro (ANSI) — BLE Bridge

ZMK firmware for the [Glorious GMMK Pro](https://www.pcgamingrace.com/products/glorious-gmmk-pro-75-barebone-black-reservation) keyboard with Bluetooth support via a nice!nano co-processor.

## Architecture

The GMMK Pro has no wireless hardware. This firmware adds BLE by inserting a **nice!nano** (nRF52840) in place of the USB daughterboard and connecting the two chips via the keyboard's internal USB cable (repurposed as UART).

```
┌─────────────────────────────────────────────────────────────────────┐
│                        GMMK Pro PCB                                 │
│                                                                     │
│  ┌──────────────────────────────────────┐                           │
│  │         STM32F303CCT6                │                           │
│  │         (Peripheral)                 │                           │
│  │                                      │                           │
│  │  Matrix 11×8 ──── PB0-PB10          │                           │
│  │                    PA0-PA4,PA8-PA10  │                           │
│  │  Encoder ──────── PC14, PC15        │                           │
│  │  AW20216S ─────── SPI1 (PA5-PA7)   │                           │
│  │                    CS: PB13, PB14   │                           │
│  │                    EN: PC13         │                           │
│  │                                      │                           │
│  │  USART3 TX ─────── PC10 ────────────┼──── D+ (USB cable)       │
│  │  USART3 RX ─────── PC11 ────────────┼──── D- (USB cable)       │
│  │  5V  ───────────────────────────────┼──── VBUS (USB cable)     │
│  │  GND ───────────────────────────────┼──── GND  (USB cable)     │
│  └──────────────────────────────────────┘                           │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
                                │
                    internal USB cable (Type-A)
                    (repurposed as UART + power)
                                │
┌─────────────────────────────────────────────────────────────────────┐
│                   nice!nano (nRF52840)                               │
│                   (Central — BLE/USB host)                          │
│                                                                     │
│   P0.06 (TX / pro_micro D1) ──── D- ──── STM32 RX (PC11)          │
│   P0.08 (RX / pro_micro D0) ──── D+ ──── STM32 TX (PC10)          │
│   VBUS ──────────────────────── VBUS ─── STM32 5V                  │
│   GND ───────────────────────── GND  ─── STM32 GND                 │
│                                                                     │
│   USB-C ──── Host PC (HID over USB or BLE)                         │
└─────────────────────────────────────────────────────────────────────┘
```

### UART Wiring (USB Cable Repurposed)

| USB Wire | Color (std) | STM32 Pin | nice!nano Pin | Direction     |
|----------|-------------|-----------|---------------|---------------|
| VBUS     | Red         | 5V        | RAW / VBUS    | STM32 → nano  |
| GND      | Black       | GND       | GND           | common        |
| D+       | Green       | PC10 (TX) | P0.08 (RX)    | STM32 → nano  |
| D-       | White       | PC11 (RX) | P0.06 (TX)    | nano → STM32  |

> **Note:** TX and RX are crossed — STM32 TX connects to nice!nano RX and vice versa.

### Data Flow

```
Keys pressed on GMMK Pro
       │
       ▼
STM32 scans matrix (11×8) at 1 ms poll rate
       │  UART 115200 baud (ZMK wired split protocol)
       ▼
nice!nano receives key events
       │
       ├──► Bluetooth LE → host (Windows/macOS/Linux/mobile)
       └──► USB-C HID → host (wired fallback)
```

RGB underglow (AW20216S) is driven directly by the STM32 via SPI1. The nice!nano does not handle LED control.

---

## Hardware

| Feature | Details |
|---------|---------|
| MCU (Peripheral) | STM32F303CCT6 (ARM Cortex-M4, 72 MHz, 256 KB Flash, 40 KB RAM) |
| MCU (Central) | nRF52840 (nice!nano v2, ARM Cortex-M4, 64 MHz, 1 MB Flash) |
| Matrix | 11 rows × 8 columns (col2row diodes) |
| Keys | 83 (ANSI layout) |
| Encoder | Alps EC11 (volume knob) |
| LED Driver | 2× AW20216S via SPI1 (RGB underglow + per-key) |
| Link | UART 115200 baud over internal USB cable |

## Pin Mapping (STM32)

### Matrix
- **Columns (outputs):** PA0, PA1, PA2, PA3, PA4, PA8, PA9, PA10
- **Rows (inputs, pull-down):** PB0–PB10

### Encoder
- **A:** PC15  **B:** PC14

### USB (on-board, for DFU flashing)
- **DM:** PA11  **DP:** PA12

### LED Driver SPI1
- **SCK:** PA5  **MOSI:** PA6  **MISO:** PA7
- **CS1:** PB13 (AW20216S #1)  **CS2:** PB14 (AW20216S #2)
- **EN:** PC13

### UART to nice!nano (USART3)
- **TX:** PC10  **RX:** PC11

## Pin Mapping (nice!nano)

- **RX (pro_micro D0 / P0.08):** ← STM32 TX (PC10)
- **TX (pro_micro D1 / P0.06):** → STM32 RX (PC11)

---

## Building

### Prerequisites

```bash
pip install west
west init -l config
west update
```

Download and install [Zephyr SDK 0.17+](https://github.com/zephyrproject-rtos/sdk-ng/releases) (ARM toolchain required).

### Build both firmwares

```bash
# STM32 Peripheral (flash via STM32 DFU)
west build -s zmk/app -b gmmk_pro/stm32f303xc/zmk \
  -DZMK_CONFIG=$(pwd)/config \
  -DZMK_EXTRA_MODULES=$(pwd)

# nice!nano Central (copy zmk.uf2 to mass storage)
west build -s zmk/app -b nice_nano/nrf52840/zmk \
  -DZMK_CONFIG=$(pwd)/config \
  -DZMK_EXTRA_MODULES=$(pwd)
```

### Flashing

**STM32 (DFU):**
```bash
# Hold BOOT0, tap RESET, then:
west flash   # or: dfu-util -a 0 -D build/zephyr/zmk.bin
```

**nice!nano (UF2):**
Double-tap RESET to enter bootloader, then copy `build/zephyr/zmk.uf2` to the `NICENANO` drive.

---

## Config (`config/west.yml`)

The AW20216S LED driver is part of ZMK mainline — no extra module is needed.

```yaml
manifest:
  remotes:
    - name: zmkfirmware
      url-base: https://github.com/zmkfirmware
  projects:
    - name: zmk
      remote: zmkfirmware
      revision: main
      import: app/west.yml
  self:
    path: config
```

---

## Status

- [x] Matrix scanning (83 keys, ANSI)
- [x] Encoder (Alps EC11)
- [x] RGB underglow / per-key (AW20216S via SPI1)
- [x] Bluetooth LE (via nice!nano central)
- [x] USB HID (via nice!nano central)
- [x] Wired split (UART 115200 baud, ZMK native)
- [x] Flash storage (NVS, last 6 KB of STM32 flash)
- [x] STM32 DFU bootloader support
- [ ] Encoder passthrough to central (sensor-bindings not yet wired)

## License

MIT
