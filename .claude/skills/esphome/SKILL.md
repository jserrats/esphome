---
name: esphome
description: Run ESPHome commands (validate, compile, upload/run, logs) for device configs in this repo. Use whenever a task involves building, validating, flashing, or checking logs for an ESPHome device config (the top-level *.yml files).
---

# Running ESPHome

ESPHome lives in a pipenv virtualenv. `pipenv` and `esphome` are **not** on PATH
in non-interactive shells, so always invoke through the Python module:

```bash
python3 -m pipenv run esphome <command> <config.yml>
```

Run from the repo root (`/Users/jserrats/personal/code/esphome`).

## Common commands

| Goal | Command |
|------|---------|
| Validate config | `python3 -m pipenv run esphome config esp-w-068.yml` |
| Compile firmware | `python3 -m pipenv run esphome compile esp-w-068.yml` |
| Upload + run (OTA) | `python3 -m pipenv run esphome run esp-w-068.yml` |
| Stream logs | `python3 -m pipenv run esphome logs esp-w-068.yml` |

After editing any config, validate with `esphome config` — a successful run ends
with `INFO Configuration is valid!`.

## Notes
- Device configs are the top-level `*.yml` files (e.g. `esp-w-068.yml`);
  `secrets.yaml` holds wifi creds.
- To render new characters in a display, add them to the relevant `font:` `glyphs:`
  string — uncompiled glyphs render as nothing.
- Shared I2S bus (mic + speaker on one `i2s_audio` id) triggers a "Parent bus is
  busy" retry loop. Use **two** `i2s_audio` instances (one TX, one RX) sharing the
  same BCLK/LRCLK/MCLK pins with `allow_other_uses: true` on each shared pin.
