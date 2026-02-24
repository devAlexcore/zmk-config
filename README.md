# Sofle RGB - PandaKB PCB - ZMK Firmware

Configuracion ZMK para un teclado split **Sofle RGB** con PCB **PandaKB** y controladores **nice_nano** (NRF52840).

## Layout actual: Colemak

```
 ┌───────────────────────────────────────────┐   ┌───────────────────────────────────────────┐
 │  `  │  1  │  2  │  3  │  4  │  5  │       │   │  6  │  7  │  8  │  9  │  0  │     │
 │ ESC │  Q  │  W  │  F  │  P  │  G  │       │   │  J  │  L  │  U  │  Y  │  ;  │ BSP │
 │ TAB │  A  │  R  │  S  │  T  │  D  │       │   │  H  │  N  │  E  │  I  │  O  │  '  │
 │SHIFT│  Z  │  X  │  C  │  V  │  B  │ MUTE  │   │  K  │  M  │  ,  │  .  │  /  │SHIFT│
 │     │CTRL │ OPT │ CMD │LOWER│SPACE│       │   │ENTER│RAISE│CTRL │ OPT │ CMD │     │
 └───────────────────────────────────────────┘   └───────────────────────────────────────────┘
```

**4 layers:** BASE (Colemak), LOWER (numeros/simbolos), RAISE (navegacion/BT), ADJUST (RGB/BT clear)

## Estructura del proyecto

```
config/
  sofle.keymap          <-- Keymap principal. Edita aqui para cambiar teclas.
  sofle.conf            <-- Configuracion del usuario (Kconfig). Gana sobre todo.
  boards/shields/sofle/
    sofle.dtsi          <-- Definicion de hardware (matriz, encoders, OLED, RGB)
    sofle_left.overlay  <-- Overlay del lado izquierdo (central)
    sofle_right.overlay <-- Overlay del lado derecho (peripheral, col-offset=6)
    sofle.conf          <-- Kconfig base del shield
    sofle_left.conf     <-- Kconfig del lado izquierdo
    sofle_right.conf    <-- Kconfig del lado derecho
```

> **IMPORTANTE:** Los archivos en `boards/shields/sofle/` son personalizados para el PCB PandaKB. NO reemplazar con archivos del Sofle estandar de ZMK.

## Compilacion local

### Requisitos

- Zephyr SDK instalado
- West workspace configurado en `.zmk/`

### Compilar lado izquierdo

```sh
cd .zmk
source zephyr/zephyr-env.sh
west build -p -s zmk/app -b nice_nano//zmk -d build/left -- \
  -DSHIELD=sofle_left \
  -DZMK_CONFIG=$(pwd)/../config \
  -DZMK_EXTRA_MODULES=$(pwd)/..
```

Firmware generado en: `.zmk/build/left/zephyr/zmk.uf2`

### Compilar lado derecho

```sh
cd .zmk
source zephyr/zephyr-env.sh
west build -p -s zmk/app -b nice_nano//zmk -d build/right -- \
  -DSHIELD=sofle_right \
  -DZMK_CONFIG=$(pwd)/../config \
  -DZMK_EXTRA_MODULES=$(pwd)/..
```

Firmware generado en: `.zmk/build/right/zephyr/zmk.uf2`

### Compilar via GitHub Actions (CI)

Hacer `git push` al repositorio. El workflow de GitHub Actions compila ambos lados automaticamente. Descarga los archivos `.uf2` desde la seccion Artifacts del Action.

### Cuando usar `-p` (pristine)

- **Siempre** usar `-p` si modificaste archivos `.dtsi` o `.overlay` (el build incremental NO detecta cambios en devicetree)
- Si solo cambiaste `sofle.keymap` o archivos `.conf`, puedes omitir `-p` para compilar mas rapido

## Flashear firmware

1. Conecta el nice_nano por USB
2. Presiona el boton de **reset dos veces rapido** (doble-press) - aparecera como unidad USB llamada `NICENANO`
3. Copia el archivo `.uf2`:
   ```sh
   cp .zmk/build/left/zephyr/zmk.uf2 /Volumes/NICENANO/   # izquierdo
   cp .zmk/build/right/zephyr/zmk.uf2 /Volumes/NICENANO/  # derecho
   ```
4. El controlador se reinicia automaticamente

### Emparejar ambas mitades (primera vez o despues de reset)

1. Flashea el **settings_reset** en ambos lados primero (ver seccion "Limpiar bonds BLE")
2. Flashea el firmware del **izquierdo** (central) y dejalo conectado por USB
3. Flashea el firmware del **derecho** (peripheral) y dejalo conectado por USB (solo para energía)
4. Espera 15-20 segundos - se emparejan automaticamente por BLE
5. Prueba las teclas del lado derecho

### Limpiar bonds BLE (si las mitades no se comunican)

Compila el firmware de reset:

```sh
cd .zmk
source zephyr/zephyr-env.sh
west build -p -s zmk/app -b nice_nano//zmk -d build/reset -- -DSHIELD=settings_reset
```

Flashea `build/reset/zephyr/zmk.uf2` en **ambos** lados, luego flashea el firmware normal en cada lado.

## Personalizar teclas

El archivo a editar es `config/sofle.keymap`. Usa sintaxis DeviceTree de ZMK.

### Cambiar una tecla individual

Busca la posicion en el layer `default_layer` y cambia el binding. Ejemplos:

```
&kp A       → letra A
&kp N1      → numero 1
&kp SPACE   → espacio
&kp BSPC    → backspace
&kp LSHFT   → shift izquierdo
&kp LGUI    → Command (Mac) / Win (Windows)
&kp LALT    → Option (Mac) / Alt (Windows)
&kp LCTRL   → Control
&none       → nada (tecla deshabilitada)
&trans      → transparente (usa el binding del layer inferior)
&mo 1       → mantener presionado para activar layer 1
```

Lista completa de keycodes: https://zmk.dev/docs/keymaps/list-of-keycodes

### Cambiar de Colemak a QWERTY

En `config/sofle.keymap`, reemplaza los bindings del `default_layer`. Cambia de:

```
// Colemak
&kp ESC  &kp Q  &kp W  &kp F  &kp P  &kp G     &kp J  &kp L  &kp U  &kp Y  &kp SEMI  &kp BSPC
&kp TAB  &kp A  &kp R  &kp S  &kp T  &kp D     &kp H  &kp N  &kp E  &kp I  &kp O     &kp SQT
&kp LSHFT &kp Z &kp X  &kp C  &kp V  &kp B     &kp K  &kp M  &kp COMMA &kp DOT &kp FSLH &kp RSHFT
```

A:

```
// QWERTY
&kp ESC  &kp Q  &kp W  &kp E  &kp R  &kp T     &kp Y  &kp U  &kp I     &kp O   &kp P     &kp BSPC
&kp TAB  &kp A  &kp S  &kp D  &kp F  &kp G     &kp H  &kp J  &kp K     &kp L   &kp SEMI  &kp SQT
&kp LSHFT &kp Z &kp X  &kp C  &kp V  &kp B     &kp N  &kp M  &kp COMMA &kp DOT &kp FSLH  &kp RSHFT
```

Luego recompila y flashea **solo el lado izquierdo** (el keymap vive en el central).

> **Nota:** Solo necesitas flashear el lado izquierdo cuando cambias el keymap. El derecho solo necesita reflash si cambias archivos `.dtsi`, `.overlay`, o `.conf` del shield.

### Agregar un combo (dos teclas simultaneas)

Agrega dentro del bloque `/ { }` en `sofle.keymap`:

```dts
combos {
    compatible = "zmk,combos";
    combo_esc {
        timeout-ms = <30>;
        key-positions = <19 20>;  // posiciones en el keymap (0-indexed)
        bindings = <&kp ESC>;
    };
};
```

### Agregar un nuevo layer

1. Define el numero del layer al inicio del archivo:
   ```
   #define MYLAYER 4
   ```
2. Agrega el layer en el bloque `keymap` (antes de las lineas `extra { status = "reserved"; }`):
   ```dts
   my_layer {
       bindings = < ... 60 bindings ... >;
   };
   ```
3. Agrega una tecla `&mo MYLAYER` o `&tog MYLAYER` en otro layer para activarlo

## Layers actuales

| # | Nombre | Activacion | Contenido |
|---|--------|-----------|-----------|
| 0 | BASE | Default | Colemak + modificadores |
| 1 | LOWER | Mantener LOWER (thumb izq) | F1-F12, numeros, simbolos (!@#$%^&*), brackets |
| 2 | RAISE | Mantener RAISE (thumb der) | Navegacion (flechas, PgUp/Dn), BT profiles, DEL |
| 3 | ADJUST | LOWER + RAISE simultaneo | BT clear, RGB controles |

## Bluetooth

- **Cambiar perfil BT:** RAISE + 1/2/3/4/5 (hasta 5 dispositivos)
- **Limpiar perfil BT actual:** RAISE + ` (tecla grave, esquina superior izquierda)
- **Limpiar todos los bonds:** ADJUST layer (LOWER + RAISE) + misma tecla

## Hardware

- **Controladores:** nice_nano (NRF52840)
- **PCB:** PandaKB (variante custom, matriz transpuesta col2row)
- **Encoder:** Boton funciona (MUTE), rotacion NO funciona (pins desconocidos en PandaKB)
- **OLED:** No soldado, deshabilitado
- **RGB:** No soldado, deshabilitado
- **Bateria:** Sin switch on/off, alimentacion solo por USB

## Troubleshooting

| Problema | Solucion |
|----------|----------|
| El lado derecho no escribe | Hacer settings_reset en ambos lados y re-flashear |
| Las teclas del derecho salen como las del izquierdo | Verificar que `sofle_right.overlay` tenga `col-offset = <6>` |
| Despues de flashear solo un lado, no se comunican | Settings_reset en ambos, re-flashear ambos |
| Teclas fantasma o teclas que no responden | Recompilar con `-p` (pristine) |
