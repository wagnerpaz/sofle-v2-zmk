# Sofle v2 → dongle conversion (ZMK)

These files turn your existing `wagnerpaz/sofle-v2-zmk` config into a 3-piece
dongle setup: the **dongle becomes the BLE central** (and talks USB to the PC),
and **both halves become peripherals**. Result: much better battery on the left
half, snappier/steadier connection, and no "wake-up" lag.

## 1. Copy these files into your repo

Drop the folders in, preserving paths (relative to repo root):

    config/sofle_dongle.keymap                                 (new)
    config/boards/shields/sofle_dongle/Kconfig.shield          (new)
    config/boards/shields/sofle_dongle/Kconfig.defconfig       (new)
    config/boards/shields/sofle_dongle/sofle_dongle.overlay    (new)
    config/boards/shields/sofle_dongle/sofle_dongle.conf       (new)
    build.yaml                                                 (REPLACES existing)

Your `config/sofle.keymap`, `config/sofle.conf` and `config/west.yml` stay
untouched. The dongle reuses `sofle.keymap` automatically (see
`sofle_dongle.keymap`), so you keep editing your layout in one place.

## 2. Push → let GitHub Actions build

Commit & push to `master`. The Actions run now produces 4 firmwares:

    sofle_dongle nice_nano_v2     -> the dongle
    sofle_left nice_nano_v2       -> left half  (now peripheral)
    sofle_right nice_nano_v2      -> right half (now peripheral)
    settings_reset nice_nano_v2   -> wipe pairings

Download the `firmware` artifact zip from the run.

## 3. Flash, in this order

For each device: double-tap RESET to mount the UF2 drive, copy the .uf2 onto it.

1. Flash **settings_reset** to ALL three devices (dongle, left, right) first.
   This clears old Bluetooth bondings — skipping it is the #1 reason pairing fails.
2. Flash **sofle_dongle** -> dongle.
3. Flash **sofle_left** -> left half.
4. Flash **sofle_right** -> right half.

## 4. Pair, in this order

Power them on one at a time, in order — this fixes which side is which
(battery reporting, etc.):

1. Dongle (plug it into USB).
2. Left half.
3. Right half.

They auto-pair to the dongle. Then connect the **dongle** to your PC over
USB (or pick a BT profile on it). You type into the dongle; the halves only
ever talk to the dongle.

## Notes

- **Superminis:** they're pin-compatible with `nice_nano_v2`, which is why every
  build targets that board. Flash via the UF2 bootloader (double-tap reset).
- **ZMK Studio** now runs through the dongle (the snippet moved to it in
  build.yaml). Open it over the dongle's USB connection.
- **Encoders & OLEDs on the halves** keep working — the halves still have their
  own hardware; only the *central role* moved to the dongle.
- **The dongle's screen is NOT configured yet.** Get this base setup typing
  first, then we add the display — the config depends on the exact panel
  (SSD1306 OLED vs. ST7789 TFT). Tell me which one it is.
- If the build ever errors on the encoder/`sensors` node, delete the
  `left_encoder`, `right_encoder` and `sensors` blocks from
  `sofle_dongle.overlay` (encoders still work via the halves) and rebuild.
