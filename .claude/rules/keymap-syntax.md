---
paths:
  - "config/sofle.keymap"
  - "config/boards/shields/sofle/sofle.keymap"
---

# ZMK Keymap Syntax Rules

This keyboard uses Colemak base layer on a Sofle RGB split (6 cols per half + encoder per half).

## Layer Definitions

Layers are defined with `#define` macros at the top:
```
#define BASE 0
#define LOWER 1
#define RAISE 2
#define ADJUST 3
```

Layer 3 (ADJUST) is a conditional layer activated when both LOWER and RAISE are held:
```
conditional_layers {
    compatible = "zmk,conditional-layers";
    adjust_layer { if-layers = <LOWER RAISE>; then-layer = <ADJUST>; };
};
```

## Row Layout Per Layer

Each layer has 5 rows of bindings:
- Row 0: 12 keys (6 left + 6 right)
- Row 1: 12 keys
- Row 2: 12 keys
- Row 3: 14 keys (6 left + encoder_left + encoder_right + 6 right)
- Thumb: 10 keys (5 left + 5 right)

Total: 60 keys + 2 encoder buttons per layer.

## Encoder Bindings

Defined per-layer in `sensor-bindings`:
```
sensor-bindings = <&inc_dec_kp C_VOL_UP C_VOL_DN &inc_dec_kp PG_UP PG_DN>;
```
First pair = left encoder, second pair = right encoder.

## Formatting

Maintain visual alignment of 6 columns per half separated by a gap for readability. Use comments to label rows when adding new layers.
