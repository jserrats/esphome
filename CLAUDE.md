# ESPHome configs

## Running ESPHome

ESPHome lives in a pipenv virtualenv. `pipenv` and `esphome` are **not** on PATH in
non-interactive shells. Invoke via the module:

```bash
python3 -m pipenv run esphome <command> <config.yml>
```

Examples:
- Validate:  `python3 -m pipenv run esphome config esp-w-068.yml`
- Compile:   `python3 -m pipenv run esphome compile esp-w-068.yml`
- Upload:    `python3 -m pipenv run esphome run esp-w-068.yml`

## Notes
- Device configs are the top-level `*.yml` files; `secrets.yaml` holds wifi creds.
- Shared I2S bus (mic + speaker on one `i2s_audio` id) triggers a "Parent bus is busy"
  retry loop. Use **two** `i2s_audio` instances (one TX, one RX) sharing the same
  BCLK/LRCLK/MCLK pins with `allow_other_uses: true` on each shared pin.
