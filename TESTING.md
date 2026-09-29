# JC8048W550C console — phase 1 bring-up checklist

Scope: screen + touch + Bambuddy connectivity on the Guition JC8048W550C
(`espoolbuddy_console_jc8048w550c.yaml`). No NFC reader, no scale.

## Build & flash

- [ ] `secrets.yaml` in the repo root (see `secrets.yaml.example` — it also needs
      `ap_fallback_password`).
- [ ] Compile: `esphome compile espoolbuddy_console_jc8048w550c.yaml`
      (CI runs the latest ESPHome release via pip; this port was built on 2026.9.0).
- [ ] First flash over USB-C at [web.esphome.io](https://web.esphome.io) with
      `.esphome/build/espoolbuddy-console-jc8048w550c/.pioenvs/espoolbuddy-console-jc8048w550c/firmware.factory.bin`.
      After that, OTA with `esphome run … --device espoolbuddy-console-jc8048w550c.local`.
- [ ] Serial log (115200 baud on the USB-C port, via the CH340) shows
      `SpoolBuddy booting...` and `WiFi connected`.

## Display

- [ ] Backlight comes on at boot.
- [ ] **Not color-inverted:** the background is near-black (dark navy, `#12121A`),
      not light grey/white.
      *If inverted:* set `invert_colors: false`.
- [ ] **Red/blue not swapped:** the active tab icon is teal (`#4ECDC4`), and AMS
      slot colors match the real filament.
      *If red and blue are swapped:* set `color_order: BGR`.
- [ ] **Not shifted:** the header touches the top edge, the bottom nav touches the bottom edge, and
      nothing is wrapped or offset horizontally. No shimmer or rolling.
      *If it jitters:* try `pclk_frequency: 15.4MHz`.

## Touch

Temporarily add this under `touchscreen:` → `on_touch:` to log coordinates:

```yaml
      - logger.log:
          level: INFO
          format: "Touch at (%d, %d)"
          args: [touch.x, touch.y]
```

- [ ] Top-left corner → roughly (0, 0)
- [ ] Top-right corner → roughly (799, 0)
- [ ] Bottom-left corner → roughly (0, 479)
- [ ] Bottom-right corner → roughly (799, 479)
      *If the axes are mirrored or swapped:* add `transform:` (`mirror_x` / `mirror_y` /
      `swap_xy`) under the `touchscreen:` entry.
- [ ] **Touch dead, and the log shows `i2c.master: I2C software timeout`:** this is
      [esphome/esphome#15356](https://github.com/esphome/esphome/issues/15356).
      GPIO19/20 double as the S3's USB pins, and the USB PHY still holds them. Apply the
      pre-`app_main` `usb_serial_jtag_ll_phy_enable_pad(false)` workaround from that issue.

## UI / Bambuddy

- [ ] All four bottom-nav tabs switch (AMS, NFC, Scale, Settings); the active icon turns teal.
- [ ] Header WiFi and cloud icons go green (backend reachable, API key accepted).
- [ ] The AMS tab shows your printer's live slots from Bambuddy (colors, materials,
      remaining %). Changing a slot on the printer updates it within about 15 s
      (`printer_poll_interval`).
- [ ] The device shows up in Bambuddy's device list as `SpoolBuddy-Console-JC8048W550C`.
- [ ] Settings has no "NFC in sleep" row and no Sound tile (expected: no NFC reader, no speaker).
      The NFC tab sits on "Waiting for tag...".

## Backlight

- [ ] Settings brightness slider changes the backlight smoothly (no flicker at low %).
- [ ] After 60 s with no touch, the backlight dims to about 20%.
- [ ] A touch while dimmed brings it back to full brightness.
- [ ] During an OTA upload the backlight goes dark, then the device reboots normally.
