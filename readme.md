<picture>
  <source media="(prefers-color-scheme: dark)" srcset="/docs/images/TOTEM_logo_dark.svg">
  <source media="(prefers-color-scheme: light)" srcset="/docs/images/TOTEM_logo_bright.svg">
  <img alt="TOTEM logo font" src="/docs/images/TOTEM_logo_bright.svg">
</picture>

# ZMK config for my TOTEM

Wireless [TOTEM](https://github.com/GEIGEIGEIST/totem) (2× Seeed XIAO nRF52840) running [ZMK](https://zmk.dev/), with a keymap ported from the default Corne (crkbd) layout. Based on [GEIGEIGEIST/zmk-config-totem](https://github.com/GEIGEIGEIST/zmk-config-totem).

- **ZMK:** `main`, pinned to commit `5b51501` in `config/west.yml` and `.github/workflows/build.yml` (keep both in sync)
- **Board:** `xiao_ble//zmk`
- **ZMK Studio:** enabled on the left half

## Layout

`·` = nothing. On Lower, Raise and Adjust, the outer keys and thumbs keep their Base function unless shown otherwise.

### Base
```
        Q    W    E    R    T        Y    U    I    O    P
        A    S    D    F    G        H    J    K    L    ;
  Ctrl  Z    X    C    V    B        N    M    ,    .    /    Esc
                 Super Lower Space   Enter Raise Shift
```

### Lower
```
        1    2    3    4    5        6    7    8    9    0
        ·    ·    ·    ·    ·        ←    ↓    ↑    →    ·
  Ctrl  ·    ·    ·    ·    ·        ·    ·    ·    ·    ·    Esc
```

### Raise
```
        !    @    #    $    %        ^    &    *    (    )
        `    ·    ·    ·    ·        -    =    [    ]    \
  Ctrl  ·    ·    ·    ·    ·        _    +    {    }    |    ~
```

### Adjust (hold Lower + Raise)
```
        BT0  BT1  BT2  BT3  BT4      ·    F7   F8   F9   F12
        CLR  USB  BLE  ·    UNLK     ·    F4   F5   F6   F11
  BOOT  RST  ·    ·    ·    ·        RST  F1   F2   F3   F10  BOOT
```

| Key | Does |
|---|---|
| BT0–BT4 | Switch to Bluetooth slot 0–4 |
| CLR | Forget the pairing on the current slot |
| USB / BLE | Send keystrokes over the cable / over Bluetooth |
| UNLK | Unlock ZMK Studio so it can save changes |
| BOOT | Enter the bootloader to flash (acts on the half it's pressed on) |
| RST | Restart (acts on the half it's pressed on) |

### Combos (Base layer only)
| Press together | Sends |
|---|---|
| Q + W | Tab |
| O + P | Backspace |
| L + ; | `'` |

Combos only fire when both keys go down within 35 ms and you haven't typed anything in the previous 150 ms, so fast rolls like "op" type normally.

Key positions for combos:
```
        0  1  2  3  4      5  6  7  8  9
       10 11 12 13 14     15 16 17 18 19
    20 21 22 23 24 25     26 27 28 29 30 31
             32 33 34     35 36 37
```

## Building

Every push builds the firmware with GitHub Actions. Open the run under **Actions**, and download **firmware** from the **Artifacts** section. It contains:

| File | Flash to |
|---|---|
| `totem_left-xiao_ble__zmk-zmk.uf2` | Left half |
| `totem_right-xiao_ble__zmk-zmk.uf2` | Right half |
| `settings_reset-xiao_ble__zmk-zmk.uf2` | Both halves, only when resetting (see Recovery) |

Test changes on a branch first; pushing a branch builds it without touching `main`.

## Flashing

1. Plug a half in by USB and **double-tap its reset button**. A USB drive appears.
2. Drag that half's `.uf2` onto the drive. The half restarts when the copy finishes.
   - macOS may report an error (e.g. -36) because the drive disappears mid-copy. That's normal.
3. Repeat for the other half.

What needs flashing:
- **Keymap-only changes:** the left half only. The keymap lives on the left half; the right half only reports key presses.
- **ZMK updates or `.conf` changes:** both halves.

Once flashed, Adjust → BOOT enters the bootloader without the reset button.

## Bluetooth

- There are **5 slots (0–4)**, one paired device per slot.
- Only the **left half** connects to devices. The right half connects to the left half on its own; never pair it.
- The selected slot is remembered across restarts and sleep.

### Pair a new device
1. Adjust → an empty slot (**BT0–BT4**). The keyboard starts advertising on that slot.
2. On the device, open the Bluetooth settings and choose **TOTEM**.
   - Linux CLI: `bluetoothctl` → `scan on` → `pair <MAC>` → `trust <MAC>` → `connect <MAC>`
3. Type something to confirm, then record the slot below.

| Slot | Device |
|---|---|
| 0 | |
| 1 | |
| 2 | |
| 3 | |
| 4 | |

### Switch devices
Adjust → the device's slot key. The switch is instant, and the other devices stay paired.

### Put a different device on a slot
1. Adjust → **BTn** to select the slot.
2. Adjust → **CLR** (the A key) clears the current slot only.
3. Remove TOTEM from the old device's Bluetooth list.
4. Pair the new device as above.

### USB vs Bluetooth
- While a USB cable connects it to a computer, keystrokes go over **USB**, even with a Bluetooth slot active.
- To charge from a computer without typing into it: Adjust → **BLE** (the D key). Adjust → **USB** (the S key) switches back.

## ZMK Studio

[ZMK Studio](https://zmk.studio/) remaps keys live, with no rebuild. Connect to the left half by USB, or by Bluetooth using the desktop app.

1. Press Adjust → **UNLK** so Studio can save changes.
2. Remap keys, or fill the two spare layers. Changes apply immediately.
3. **Copy every change you keep into `config/totem.keymap`.** Studio can't export, so this file is the only backup.
4. After flashing new firmware, click **Restore Stock Settings** in Studio so the keyboard runs exactly what's in the file.

Studio changes survive flashing, and while they exist, edits to the layers in `totem.keymap` are ignored. That's why steps 3 and 4 matter.

Studio can't change combos, the Lower+Raise → Adjust rule, or `config/totem.conf`; those always come from this repo.

## Settings (`config/totem.conf`)
- Deep sleep after 15 minutes idle. A keypress wakes it; that key is lost.
- The right half's battery level is reported to the host too.

## Recovery

| Problem | Fix |
|---|---|
| One device won't reconnect | Select its slot, press CLR, remove TOTEM on the device, pair again |
| The halves don't connect to each other, or every device misbehaves | Remove TOTEM on every device. Flash `settings_reset` to **both** halves, then the normal firmware to each, then pair again |
| A half seems dead | Double-tap reset and flash its firmware again, or a saved older `.uf2` |
| Keymap file changes don't show up | Studio changes are overriding them: click **Restore Stock Settings** in Studio |

`settings_reset` erases Bluetooth pairings **and** Studio changes. Copy any Studio changes into `totem.keymap` first.

**Keep a known-good firmware:** GitHub deletes build artifacts after 90 days. Before flashing anything new, double-tap reset and copy `CURRENT.UF2` off each half's drive, or keep the last working `.uf2` files locally.
