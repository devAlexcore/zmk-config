---
name: flash-firmware
description: Guide user through flashing .uf2 firmware to nice_nano controllers
argument-hint: [left|right|both]
user-invocable: true
allowed-tools: Bash, Read
---

Guide the user through flashing firmware to the nice_nano v2.

## Flash Procedure

The nice_nano enters bootloader (UF2 mass storage) by **double-tapping the reset button**.

1. Check that the `.uf2` files exist:
   - Left: `.zmk/build/left/zephyr/zmk.uf2`
   - Right: `.zmk/build/right/zephyr/zmk.uf2`

2. For the requested half ($ARGUMENTS, default "both"):

   a. Tell the user: "Double-tap the reset button on the **[left/right]** half"
   b. Wait for the user to confirm the drive appeared
   c. Find the mounted volume: `ls /Volumes/ | grep -i nice`
   d. Copy the firmware: `cp .zmk/build/[left|right]/zephyr/zmk.uf2 /Volumes/NICENANO/`
   e. The controller will auto-reboot after copy completes

3. If flashing both halves, do left first (central), then right (peripheral).

## Troubleshooting

- If volume doesn't appear: hold reset for 5+ seconds, or try a different USB cable
- If keyboard doesn't pair after flash: flash `settings_reset` first, then reflash normal firmware
- Build settings_reset: `west build -p -s zmk/app -b nice_nano//zmk -d build/reset -- -DSHIELD=settings_reset`
