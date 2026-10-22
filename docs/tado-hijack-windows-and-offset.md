# Window sensors and offset calibration with Tado Hijack (no blueprint)

If you use the [Tado Hijack](https://github.com/banter240/tado_hijack) integration, you do not need
`temp_sensor_calibration.yaml` any more. Tado Hijack can already:

- turn a room **off while a window is open** and resume the schedule when it closes, and
- **calibrate the valve offset** from an external temperature sensor.

The only thing it does not do on its own is wait a few minutes before reacting, or combine several
window sensors for one room. A template binary sensor with a delay covers both. Everything below is
done from the Home Assistant UI, no YAML.

## Why not a normal automation

Tado zones usually have a default manual-control duration (for example *Timer, 1 hour*). An
automation that calls `climate.set_hvac_mode: off` creates that kind of overlay, so the valve
**turns itself back on after one hour even if the window is still open**. Tado Hijack's window
handler sends an off without an end time instead, and resumes the schedule when the window closes.

## 1. Create one delayed window sensor per room

*Settings → Devices & services → Helpers → Create helper → Template → Template a binary sensor*

| Field | Value |
|---|---|
| Name | `Tado window <room>` |
| State template | see below |
| Device class | Window |
| Advanced options → Delay on | `00:03:00` |
| Advanced options → Delay off | `00:03:00` |

One contact sensor in the room:

```jinja
{{ is_state('binary_sensor.bedroom_window', 'on') }}
```

Several openings that share one valve (living room with a door and some windows): the room counts as
open while **any** of them is open, and as closed only when **all** of them have been closed for the
delay.

```jinja
{{ is_state('binary_sensor.front_door', 'on')
   or is_state('binary_sensor.kitchen_window', 'on')
   or is_state('binary_sensor.living_window_1', 'on')
   or is_state('binary_sensor.living_window_2', 'on') }}
```

With a delay, airing a room for a minute or walking through a door changes nothing, and a window
opened and closed several times does not make the valve switch on and off. A good starting point is
**5 minutes on** (window open) and **2 minutes off** (window closed).

### Different rules for one room

A template binary sensor has a single *Delay on*. If a room needs two rules, for example *"turn off
after 15 minutes when two windows are open, or after one hour when only one is"*, create one helper
per rule and a third one that combines them:

| Helper | State template | Delay on | Delay off |
|---|---|---|---|
| `Tado window living - 2 open` | `{{ ['binary_sensor.w1','binary_sensor.w2','binary_sensor.w3'] \| select('is_state','on') \| list \| count >= 2 }}` | `00:15:00` | – |
| `Tado window living - 1 open` | same list, `count >= 1` | `01:00:00` | – |
| `Tado window living` | `{{ is_state('binary_sensor.tado_window_living_2_open','on') or is_state('binary_sensor.tado_window_living_1_open','on') }}` | – | `00:02:00` |

Only the last one is linked to Tado Hijack.

## 2. Link the sensor to the Tado Hijack zone

On each Tado Hijack zone device:

| Entity | Value |
|---|---|
| `select.<zone>_window_sensor` | `binary_sensor.tado_window_<room>` |
| `select.<zone>_window_mode` | `direct` |

`direct`: open → zone off with no end time; closed → schedule resumes (if the room was in manual
mode, the manual setpoint is sent back). `timeout` instead resumes after the zone's own open-window
timeout even if the window is still open. Hijack keeps the state across a Home Assistant restart.

## 3. Faster: local control through HomeKit (V3 / V3+)

By default the off and the resume go through the Tado cloud. They work, but there is a delay: resumes
are batched (*Resume schedule batch window*, 120 s by default) and the core Tado integration only
shows the change at its next poll.

If you have a V3 bridge, pair it with Home Assistant through the **HomeKit Device** integration.
Tado Hijack finds the valves by serial number on its own (look for *"AvailabilityTracker tracking N
local climate entities"* in the log). From then on an open window turns the valve off **locally, in
seconds, without a cloud call**, and turns it off again if someone switches it on while the window is
still open.

- A HomeKit accessory can only be paired with one controller. If the bridge is in Apple's Home app,
  remove it there first (*Accessory settings → Remove accessory*). The Tado app and its schedules are
  not affected.
- Do not remove the HomeKit Device integration afterwards: Tado Hijack relies on it.
- If you stay on the cloud, you can set *Resume schedule batch window* to `0` to resume without
  waiting.

## 4. Offset calibration (optional)

Valves read the temperature right next to the radiator. If you have a better-placed sensor:

| Entity | Value |
|---|---|
| `select.<zone>_temperature_source` | your room sensor (e.g. `sensor.bedroom_temperature`) |
| `select.tado_<home>_offset_cal_interval` | `threshold` (or a fixed interval such as `3h`) |
| `number.tado_<home>_offset_cal_spread_threshold` | minimum difference before writing, e.g. `1` |

Leave `temperature_source` on `device` for rooms where the valve reading is good enough. Calibration
is skipped while that room's window sensor is open, and resumes a few minutes after it closes.

## 5. Turn the old blueprint off

Disable (do not delete) the automations created from `temp_sensor_calibration.yaml`. Its window logic
and its offset writes would otherwise fight with Tado Hijack.

## Examples (5 minutes on, 2 minutes off)

| What happens | Result |
|---|---|
| Window open for 2 minutes, then closed | nothing |
| Window open at 09:00, closed at 10:30 | valve off at 09:05, back on schedule at 10:32 |
| Living room with the "2 open / 1 open" rule: one window open all evening | off after one hour |
| Living room: two windows open at 15:00, both closed at 15:40 | off at 15:15, back on schedule at 15:42 |
| Home Assistant restarts while a window is open | valve stays off, resumes when the window closes |
