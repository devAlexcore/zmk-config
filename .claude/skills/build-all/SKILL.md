---
name: build-all
description: Build firmware for both halves of the Sofle keyboard locally
user-invocable: true
allowed-tools: Bash, Read
---

Build both halves sequentially:

```sh
cd /Users/devcore/Documents/keyboars/zmk-config/zmk-config/.zmk && \
source zephyr/zephyr-env.sh && \
echo "=== Building LEFT ===" && \
west build -s zmk/app -b nice_nano//zmk -d build/left -- \
  -DSHIELD=sofle_left \
  -DZMK_CONFIG=/Users/devcore/Documents/keyboars/zmk-config/zmk-config/config \
  -DZMK_EXTRA_MODULES=/Users/devcore/Documents/keyboars/zmk-config/zmk-config && \
echo "=== Building RIGHT ===" && \
west build -s zmk/app -b nice_nano//zmk -d build/right -- \
  -DSHIELD=sofle_right \
  -DZMK_CONFIG=/Users/devcore/Documents/keyboars/zmk-config/zmk-config/config \
  -DZMK_EXTRA_MODULES=/Users/devcore/Documents/keyboars/zmk-config/zmk-config
```

Add `-p` flag after each `west build` only if user requests a clean/pristine build.

After builds complete, report a summary table:

| Half | Status | Flash | RAM | UF2 Path |
|------|--------|-------|-----|----------|
| Left | ... | ...% | ...% | `.zmk/build/left/zephyr/zmk.uf2` |
| Right | ... | ...% | ...% | `.zmk/build/right/zephyr/zmk.uf2` |
