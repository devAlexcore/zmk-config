---
name: build-left
description: Build firmware for the left half of the Sofle keyboard locally
user-invocable: true
allowed-tools: Bash, Read
---

Build the left half firmware locally:

```sh
cd /Users/devcore/Documents/keyboars/zmk-config/zmk-config/.zmk && \
source zephyr/zephyr-env.sh && \
west build -s zmk/app -b nice_nano//zmk -d build/left -- \
  -DSHIELD=sofle_left \
  -DZMK_CONFIG=/Users/devcore/Documents/keyboars/zmk-config/zmk-config/config \
  -DZMK_EXTRA_MODULES=/Users/devcore/Documents/keyboars/zmk-config/zmk-config
```

Add `-p` flag after `west build` only if user requests a clean/pristine build.

After build completes, report:
1. Success or failure
2. Memory usage (FLASH and RAM percentages)
3. Location of `.uf2` file: `.zmk/build/left/zephyr/zmk.uf2`
4. Any warnings worth noting
