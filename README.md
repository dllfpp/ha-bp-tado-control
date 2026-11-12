# Blueprint: Tado TRV offset calibration

This blueprint allows you to dynamically update the temperature offset of a Tado climate entity (or another climate device) based on a Zigbee sensor's readings, while providing control to enable or disable the automation based on the climate entity's operating mode (specifically, when in HEAT mode).

> **Tado Hijack users:** it can calibrate offsets and handle windows natively, see
> [docs/tado-hijack-windows-and-offset.md](docs/tado-hijack-windows-and-offset.md). This blueprint supports
> both the core Tado integration and Tado Hijack (pick its offset entity).

## What it does

- **Offset calibration:** at the chosen interval it compares the valve reading with a
  separate room sensor and, if they differ by more than the threshold, corrects the valve
  temperature offset. Only inside the time window and, optionally, only while heating.
- **Battery saving:** offset writes are rate-limited (default one every 2 hours per valve),
  because each write makes the valve motor close and reopen.
- **Window / alarm contacts (optional):** pick one or more door/window contacts (for example
  from an alarm system). If any stays open for the chosen delay the valve is switched off;
  when all are closed it goes back to `auto` (or `heat`). Leave it empty if your integration
  already handles windows.

## Inputs

| Input | Required | Description |
|---|---|---|
| Valve | yes | Climate entity of the Tado TRV |
| Room temperature sensor | yes | Separate temperature sensor the valve must match |
| Offset entity | no | Tado Hijack number entity for the offset; empty = core Tado action `tado.set_climate_temperature_offset` |
| Threshold | no (1 °C) | Minimum difference before correcting |
| Check every | no (15 min) | How often the comparison runs |
| Minimum hours between offset writes | no (2 h) | Rate limit (applies with the offset entity) |
| Active from / until | no (06:00–23:00) | Time window |
| Only while heating | no (on) | Correct only while the valve is heating |
| Window / alarm contacts | no | One or more `binary_sensor` contacts |
| Window open for | no (2 min) | Delay before switching the valve off |
| Mode when windows close | no (auto) | `auto` or `heat` |

## Installation

[![Open your Home Assistant instance and show the blueprint import dialog.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fdllfpp%2Fha-bp-tado-control%2Fblob%2Fmain%2Ftemp_sensor_calibration.yaml)

Or: Settings → Automations & scenes → Blueprints → Import blueprint, and paste
`https://github.com/dllfpp/ha-bp-tado-control/blob/main/temp_sensor_calibration.yaml`.
Then create one automation per valve.

Requires Home Assistant 2024.10 or newer.

## License

MIT, see [LICENSE](LICENSE).
