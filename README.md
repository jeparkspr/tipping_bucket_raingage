# ESPHome Tipping Bucket Rain Gauge (0.01 in/tip)

This project implements a tipping bucket rain gauge using an ESP32 DevKit V1 and ESPHome. 
Each bucket tip represents **0.01 inches** of rainfall. The device counts tips, calculates 
cumulative rainfall, allows manual reset/set from Home Assistant, and exposes data for both 
short-term rolling totals and calendar-based summaries.

## Features

- Reed switch input on GPIO18 (software debounced, persistent count across reboots)
- Cumulative rainfall total (inches) with `state_class: total_increasing`
- Manual set/reset of rainfall total via Home Assistant number entity
- Rolling rainfall totals (last 5, 15, 30, 60 minutes) via HA `statistics` platform
- Calendar-based totals (today, this week, this month) via HA `utility_meter`
- Web server enabled for local status viewing
- Static IP and Wi-Fi connection logging

## Hardware

- ESP32 DevKit V1 (30-pin or 36-pin)
- Tipping bucket rain gauge with **normally open (NO) reed switch** (magnet closes switch on tip)
- Recommended: 10 kΩ resistor (external pull-up from 3.3V to GPIO18 for noise immunity)

### Wiring
```
                ESP32                 ┋                 Rain Gauge
----------------------------------------------------------------------------------
                                      ┋    
3.3V ══════════[10kΩ]═════════╗       ┋
                              ║       ┋
GPIO18 (input) ═══════════════╩═══════════════════╗
                                      ┋            /   Reed Switch (closes on tip)
GND ══════════════════════════════════════════════╝
                                      ┋
```

- Switch leg 1 → GPIO18
- Switch leg 2 → GND
- Internal pull-up enabled (`INPUT_PULLUP`); external 10 kΩ optional but recommended outdoors

## Software

All configuration files are in the root of this repository:

- `raingage.yaml` → ESPHome device configuration (flashed to ESP32)
- `sensors.yaml` → Home Assistant rolling raw totals (5/15/30/60 min) using `statistics` platform
- `templates.yaml` → Home Assistant rolling totals for display (5/15/30/60 min) using `statistics` platform
- `utility_meters.yaml` → Home Assistant calendar totals (daily/weekly/monthly) using `utility_meter`

These files are referenced from `configuration.yaml` like so:

```
# In configuration.yaml
sensor: !include sensors.yaml
utility_meter: !include utility_meters.yaml
```

## Home Assistant Setup

- Add the ESPHome device via the ESPHome add-on/integration.
- Include the YAML files in configuration.yaml (as shown above).
- Validate configuration and restart Home Assistant.
- Optional display tweaks:
Set display precision = 2 on each utility_meter entity (via entity settings page) for clean 0.01-inch display.
Manually reset calendar totals using utility_meter.calibrate service (value: 0).


### Exposed Entities (examples)

- sensor.raingage_rainfall_total – Cumulative rainfall (inches)
- sensor.raingage_rain_tips_total – Raw tip count
- number.raingage_set_rainfall_total – Set/reset total rainfall
- sensor.rainfall_total_5_min / 15_min / 30_min / 60_min – Rolling totals
- sensor.rainfall_today / this_week / this_month – Calendar totals

### Calibration

- Each tip = 0.01 inches
- Scaling factor appears in two places:
- rainfall_total lambda: * 0.01
- Reset lambda: * 100.0 (inches → tips)

Change both if your gauge uses a different calibration.

### Future Ideas

- Add heavy rain alerts (automation when 5-min or 15-min total exceeds threshold)
- Daily/weekly graphs in Lovelace
- Battery + deep sleep version for remote deployment
- MQTT publishing for non-HA integrations

## License
MIT – free to use, modify, and share.
