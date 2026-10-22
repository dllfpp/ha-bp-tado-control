# Blueprint: Update Entity Offset with Switch Control
## Tado/Zigbee/Working Hours/ON-OFF Switch

This blueprint allows you to dynamically update the temperature offset of a Tado climate entity (or another climate device) based on a Zigbee sensor's readings, while providing control to enable or disable the automation based on the climate entity's operating mode (specifically, when in HEAT mode).

> **Using the [Tado Hijack](https://github.com/banter240/tado_hijack) integration?** You no longer need
> this blueprint: Tado Hijack handles both window sensors (with no 1-hour expiry) and offset
> calibration natively. See [docs/tado-hijack-windows-and-offset.md](docs/tado-hijack-windows-and-offset.md).
> The blueprint stays here for setups with the core Tado integration only.

## Use Case:
You may want to adjust the temperature offset on your climate entity based on a more accurate or strategically placed sensor. This blueprint enables you to automate that adjustment within a specified time range and only when the heating system is on.

## How It Works:
- `Sensor Monitoring:` You specify a Zigbee temperature sensor whose state will be monitored.
- `Entity Update:` You choose a climate entity (like Tado) whose temperature offset will be adjusted.
- `Threshold Control:` You can set a maximum allowed temperature difference between the monitored sensor and the climate entity before triggering an update.
- `Time Range:` You can define start and stop times for when the automation should be active.
- `Heat Mode Control:` There's an optional toggle to enable or disable this automation based on whether the climate entity is in HEAT mode. If the toggle is off, the automation will run regardless of the heating mode.

## Inputs:
- `Zigbee Sensor to Monitor:` The temperature sensor whose readings will determine the temperature offset.
- `Tado Entity to Update:` The climate entity (Tado or similar) whose offset will be updated.
- `Threshold:` The temperature difference required to trigger an update.
- `Start Time:` Time when the automation starts.
- `End Time:` Time when the automation ends.
- `Enable Automation Only When Heating:` Boolean to control whether the automation only runs when the climate entity is in HEAT mode.

## Example:
You have a Zigbee sensor in your living room and want to adjust the offset on your Tado thermostat based on the difference between the Tado’s temperature reading and the more accurate one from the Zigbee sensor. This automation allows you to apply this adjustment automatically, but only during specific times (e.g., 6 AM to 10 PM) and only when the Tado is heating.

## Trigger & Condition:
The automation triggers every 15 minutes and checks the time of day and the heating mode condition (if enabled). If the difference between the monitored sensor and the Tado’s temperature exceeds the threshold, the Tado’s offset is updated.

## Action:
The action recalculates and updates the offset value based on the temperature difference between the selected Zigbee sensor and the climate entity.

## License

MIT, see [LICENSE](LICENSE).
