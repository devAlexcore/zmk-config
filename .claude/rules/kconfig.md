---
paths:
  - "config/sofle.conf"
  - "config/boards/shields/sofle/*.conf"
---

# Kconfig Rules

## Merge Order (later wins)

1. `config/boards/shields/sofle/sofle.conf` — Shield base
2. `config/boards/shields/sofle/sofle_left.conf` — Left overrides
3. `config/boards/shields/sofle/sofle_right.conf` — Right overrides
4. `config/sofle.conf` — User overrides (final word)

## Current Disabled Features (not soldered)

These are explicitly set to `n` and should stay off unless hardware is added:
- `CONFIG_ZMK_DISPLAY` — OLED
- `CONFIG_ZMK_RGB_UNDERGLOW` — RGB LEDs

## Key Settings

- `CONFIG_ZMK_KEYBOARD_NAME="Sofle_RGB"` — BLE advertised name
- `CONFIG_BT_CTLR_TX_PWR_PLUS_8=y` — Max BLE range
- `CONFIG_ZMK_SLEEP=y` — Deep sleep enabled
- `CONFIG_ZMK_IDLE_TIMEOUT=600000` — 10 min to idle
- `CONFIG_ZMK_IDLE_SLEEP_TIMEOUT=900000` — 15 min to deep sleep
- `CONFIG_EC11=y` — Encoder driver

## When Editing

- To enable a feature, set it in `config/sofle.conf` (user level) — it overrides everything
- To change per-half behavior, edit `sofle_left.conf` or `sofle_right.conf`
- Don't edit `sofle.conf` in the shield directory unless changing the base defaults for both halves
