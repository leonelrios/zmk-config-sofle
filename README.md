# Wireless Sofle · ZMK config

[ZMK](https://zmk.dev) configuration for my wireless split Sofle: two nRF52840 boards (SuperMini, `nice_nano_v2` compatible), a hand-wired matrix, and a 128×32 OLED display on the right half.

![My Sofle](docs/sofle.jpg)

## Keymap

There are two layers. Layer 0 is for normal typing, and layer 1 is active **while holding** the `MO 1` key (right thumb).

### Layer 0 · BASE

![BASE layer](docs/keymap-base.svg)

QWERTY with a number row on top. All modifiers (`Ctrl`, `Alt`, `Gui`, `Shift`) sit on the left thumb cluster. The right thumb cluster has `Enter`, `Space`, `Backspace`, the layer key and `Esc`.

### Layer 1 · MO

![MO layer](docs/keymap-mo.svg)

| Area | What it does |
|---|---|
| Top row | Shifted symbols (`! @ # $ % ^ & * ( ) _ +`) without pressing Shift |
| Left side | `F1`–`F10` |
| Right side | Inverted-T arrows (`↑ ← ↓ →`), plus `Home`, `End`, `PgUp`, `PgDn` and `Del` |
| `BT 0` / `BT 1` / `BT 2` | Select a Bluetooth profile (up to 3 paired devices) |
| `BT clear` | Clear the pairing of the **active** profile |
| `Vol−` / `Vol+` | System volume |
| `OLED power` | Toggle external power (`ext_power`), handy for cutting power to the OLED to save battery |
| `Boot (left)` | Put the **left** half into bootloader mode for flashing |
| `Boot (right)` / `Reset (right)` | Bootloader or reset for the **right** half |
| `▽` | Transparent: falls through to the BASE layer |

> In ZMK, `&bootloader` and `&sys_reset` act on the half where the key physically is. That's why there is a Boot key on each side.

The keymap uses **US** layout keycodes. If your OS uses a different layout (e.g. Latin American Spanish), some symbols will come out differently from what the diagrams show.

## Hardware

| | Left | Right |
|---|---|---|
| Board | nRF52840 SuperMini (`nice_nano_v2`) | nRF52840 SuperMini (`nice_nano_v2`) |
| Role | **Central**: connects to the computer via USB or Bluetooth | Peripheral: connects to the left half |
| Display | — | SSD1306 128×32 OLED over I²C (SDA `P0.29`, SCL `P1.13`) |
| Matrix | 5 rows × 6 columns | 5 rows × 6 columns |

Row 4 holds the thumb keys: 5 per side (C0 is unused on the left and C5 on the right). Diodes are `col2row`, and debounce is 20 ms for both press and release.

Since the keyboard is hand-wired, the pins don't match an official Sofle PCB, and each half has its own:

<details>
<summary>Matrix pins</summary>

**Left** (`boards/shields/mi_sofle/mi_sofle_left.overlay`)

| | C0 | C1 | C2 | C3 | C4 | C5 |
|---|---|---|---|---|---|---|
| Column | P1.15 | P1.01 | P0.11 | P0.24 | P1.06 | P0.29 |

| | R0 | R1 | R2 | R3 | R4 |
|---|---|---|---|---|---|
| Row | P0.31 | P0.02 | P1.11 | P0.09 | P1.07 |

**Right** (`boards/shields/mi_sofle/mi_sofle_right.overlay`)

| | C0 | C1 | C2 | C3 | C4 | C5 |
|---|---|---|---|---|---|---|
| Column | P0.09 | P0.10 | P1.11 | P1.07 | P0.20 | P0.08 |

| | R0 | R1 | R2 | R3 | R4 |
|---|---|---|---|---|---|
| Row | P0.06 | P0.17 | P0.22 | P1.06 | P1.01 |

</details>

## Display and battery

The OLED uses the [zmk-nice-oled](https://github.com/mctechnology17/zmk-nice-oled) module (pinned to `v0.0.2`) with the vertical screen. It shows a static image instead of an animation, which uses less power, and the image is drawn off-screen so no icon is visible. To bring it back, remove `CONFIG_NICE_OLED_WIDGET_ANIMATION_PERIPHERAL_CUSTOM_X=300` from `config/mi_sofle_right.conf`.

Power saving (`config/mi_sofle.conf`):

- The display turns off after **30 s** of inactivity.
- The keyboard goes into deep sleep after **15 min** and wakes up on any key press.

## Repo structure

```
boards/shields/mi_sofle/   Custom shield: matrix, pins and OLED
config/mi_sofle.keymap     Keymap
config/mi_sofle.conf       Shared options (sleep, timeouts)
config/mi_sofle_right.conf OLED options (right half only)
config/west.yml            ZMK (v0.3) and zmk-nice-oled versions
build.yaml                 Firmware targets built by GitHub Actions
```

## Building and flashing

1. Every push builds the firmware on **GitHub Actions**. Download the `.zip` from *Artifacts* in the Actions tab.
2. Plug one half in via USB and enter the bootloader: double-tap the reset button, or press `MO 1` + that half's `Boot` key.
3. A USB drive shows up. Copy the matching `.uf2` onto it:
   - `mi_sofle_left-nice_nano_v2-zmk.uf2` → left half
   - `mi_sofle_right nice_oled-nice_nano_v2-zmk.uf2` → right half
4. Repeat with the other half.

If the halves won't connect to each other, flash `settings_reset-nice_nano_v2-zmk.uf2` onto **both** halves, then flash the regular firmware on each side again.

If your board isn't a nice!nano v2 or compatible, change `board:` in `build.yaml`.
