---
paths:
  - "config/boards/shields/sofle/**"
---

# PandaKB Hardware Rules

## CRITICAL: Transposed Matrix

This PCB has rows and columns physically swapped vs standard Sofle. The shield at `config/boards/shields/sofle/` is a custom override — **never replace with upstream ZMK shield files**.

- Diode direction: `col2row` (standard Sofle = `row2col`)
- Matrix transform: `columns = <12>`, `rows = <6>` (6 cols per half)
- Row GPIOs (6): pins 19, 18, 15, 14, 16, 10 → physical columns
- Col GPIOs (6): pins 9, 8, 7, 6, 5, **4** → physical rows
- **Kscan row 0 (pin 19)** is not triggered by any physical key

## Left Side Matrix — Empirically Confirmed (2026-02-24)

Physical key → kscan RC mapping (left side):

```
Numbers row: RC(1,5)  RC(1,4) RC(1,3) RC(1,2) RC(1,1) RC(1,0)
QWFP row:    RC(2,5)  RC(2,4) RC(2,3) RC(2,2) RC(2,1) RC(2,0)
ARST row:    RC(3,5)  RC(3,4) RC(3,3) RC(3,2) RC(3,1) RC(3,0)
ZXCV row:    RC(4,5)  RC(4,4) RC(4,3) RC(4,2) RC(4,1) RC(4,0)
Encoder btn: RC(5,0)
Thumb row:   RC(5,5)  RC(5,4) RC(5,3) RC(5,2) RC(5,1)
```

Key findings:
- Physical rows map to kscan rows 1-5 (row 0 unused)
- Columns are reversed: leftmost physical col → highest kscan col index
- The 6th col-gpio (pin 4) was discovered empirically — enables the leftmost column
- Thumb row keys: CTRL, Option, Command, LOWER, Space (user preference)

## Right Side Matrix — CONFIRMED WORKING (2026-02-24)

Right side uses cols 6-11 via `col-offset = <6>` in `sofle_right.overlay`. The right PCB is physically flipped, so columns go in **ascending** order (opposite to left side).

```
Numbers row: RC(1,6)  RC(1,7)  RC(1,8)  RC(1,9)  RC(1,10) RC(1,11)
JLUY row:    RC(2,6)  RC(2,7)  RC(2,8)  RC(2,9)  RC(2,10) RC(2,11)
HNEI row:    RC(3,6)  RC(3,7)  RC(3,8)  RC(3,9)  RC(3,10) RC(3,11)
KMCOMMA row: RC(4,6)  RC(4,7)  RC(4,8)  RC(4,9)  RC(4,10) RC(4,11)
Encoder btn: RC(5,6)
Thumb row:   RC(5,7)  RC(5,8)  RC(5,9)  RC(5,10) RC(5,11)
```

Key findings:
- Left side columns: 5→0 (descending, physical left to right)
- Right side columns: 6→11 (ascending, physical left to right) — PCB is flipped
- `col-offset = <6>` is **required** in `sofle_right.overlay` — without it, the peripheral reports cols 0-5 (same as left) and all right-side keys produce left-side bindings
- The offset is applied by the peripheral's matrix transform at compile time, NOT by the split BLE transport

## Encoder — Rotation NOT WORKING

The encoder **button press** works via the key matrix at RC(5,0).

The encoder **rotation** does NOT work. The following pin pairs were tested for EC11 A/B and NONE responded:
- Pins 20/21 (standard Sofle)
- Pins 21/20 (swapped)
- Pins 0/1
- Pins 1/2
- Pins 2/3
- Pins 0/3
- Pins 0/21

A full matrix diagnostic (6×6 grid covering all kscan positions including row 0) was also tested — rotating the encoder produced NO matrix events. The rotation is neither on direct GPIO pins nor routed through the key matrix.

**Possible causes:** The encoder rotation may connect to pins not accessible via the pro_micro pinout, or may require PandaKB-specific routing information that is unavailable (no schematic exists).

**Current status:** Encoder rotation is disabled. Only the mute button (press) is functional.

## Peripherals

- OLED: SSD1306, 128x32, I2C @ 0x3c. **Disabled** (not soldered). I2C bus set to `status = "disabled"`.
- RGB: WS2812, 36 LEDs, SPI3. **Disabled** (not soldered).
- Encoder button: Works via matrix. Rotation: disabled (see above).

## Pin Usage Summary

| Pin | Function | Status |
|-----|----------|--------|
| 19 | row-gpio 0 | In kscan but never fires — purpose unknown |
| 18 | row-gpio 1 | Physical column — active |
| 15 | row-gpio 2 | Physical column — active |
| 14 | row-gpio 3 | Physical column — active |
| 16 | row-gpio 4 | Physical column — active |
| 10 | row-gpio 5 | Physical column — active |
| 9 | col-gpio 0 | Physical row — active |
| 8 | col-gpio 1 | Physical row — active |
| 7 | col-gpio 2 | Physical row — active |
| 6 | col-gpio 3 | Physical row — active |
| 5 | col-gpio 4 | Physical row — active |
| 4 | col-gpio 5 | Physical row — active (6th, discovered empirically) |
| 21, 20 | Encoder (EC11) | Defined but rotation doesn't work |
| 0, 1, 2, 3 | Free | Tested for encoder — no response |

## Shield File Roles

- `sofle.dtsi` — Shared hardware: matrix, encoders, OLED, RGB
- `sofle_left.overlay` — Includes dtsi (encoder rotation disabled)
- `sofle_right.overlay` — Includes dtsi + `col-offset=6` + enables right encoder
- `Kconfig.defconfig` — Left=central role, split config, display driver setup
- `Kconfig.shield` — Shield detection macros
- `sofle.conf` — Base Kconfig (all features ON)
- `sofle_left.conf` / `sofle_right.conf` — Per-half overrides
