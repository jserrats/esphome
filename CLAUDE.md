# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running ESPHome

ESPHome lives in a pipenv virtualenv. `pipenv` and `esphome` are **not** on PATH in
non-interactive shells. Invoke via the module, from the repo root:

```bash
python3 -m pipenv run esphome <command> <config.yml>
```

- Validate:  `python3 -m pipenv run esphome config esp-w-068.yml` (ends with `INFO Configuration is valid!`)
- Compile:   `python3 -m pipenv run esphome compile esp-w-068.yml`
- Upload OTA + run: `python3 -m pipenv run esphome run esp-w-068.yml`
- Stream logs: `python3 -m pipenv run esphome logs esp-w-068.yml`

For the initial USB flash (no OTA yet), pass a device, e.g. `esphome run <config> --device /dev/ttyACM0`.

## Architecture

This repo is a flat collection of **standalone** ESPHome device configs — there are no
shared packages or `!include` files. Each top-level `*.yml` fully describes one device;
edits to one config never affect another.

- **Naming**: the standard is a bare `esp-w-<id>.yml` (e.g. `esp-w-009.yml`), where
  `<id>` is the 3-digit physical device ID. This must match the node `name:` in the
  `esphome:` block and the `use_address:` hostname (`esp-w-<id>.iot`) — file, node name,
  and mDNS hostname stay in lockstep. An older convention embedded the location as
  `<floor>-<room>-esp-w-<id>.yml` (e.g. `0f-studio-esp-w-009.yml`); migrate any such file
  to the bare form. Renaming the node `name:` changes the device identity in Home
  Assistant (old entities go unavailable, new ones are created), and the new
  `esp-w-<id>.iot` hostname only resolves after the device is reflashed — so the migration
  flash must target the device's current IP with `esphome run <cfg> --device <ip>`.
- **Multiple chip families / frameworks coexist**, so config idioms differ per file:
  ESP8266 (`esp01_1m`), ESP32-C3 (arduino framework), ESP32-S3 (esp-idf framework). Match
  the target chip's conventions when editing — don't assume all files look alike.
- **Home Assistant is the hub.** Every device sets `api:` and reaches HA over WiFi at
  `use_address: <name>.iot` (or a static IP). Devices pull HA state via `homeassistant`
  platform `sensor`/`text_sensor` entities and render them; a recurring pattern is showing
  `input_text.notification_details` on an attached LCD/display.
- **All configs share `secrets.yaml`** (`wifi_ssid`, `wifi_pass`).
- `config/` holds display fonts (`.ttf`); `deprecated/` holds retired configs (ignore unless
  asked); `.esphome/` is the gitignored build cache.

`esp-w-068.yml` is the most complex device: an ESP32-S3 voice assistant with an ST7305
reflective-LCD status screen (external component), ES8311 DAC + ES7210 ADC audio, and a
HA voice pipeline. Use it as the reference for advanced patterns.

## Gotchas

- **Display glyphs must be pre-declared.** To render a new character, add it to the relevant
  `font:` `glyphs:` string — glyphs not compiled in render as nothing.
- **Shared I2S bus** (mic + speaker on one `i2s_audio` id) triggers a "Parent bus is busy"
  retry loop. Use **two** `i2s_audio` instances (one TX, one RX) sharing the same
  BCLK/LRCLK/MCLK pins with `allow_other_uses: true` on each shared pin (see `esp-w-068.yml`).
