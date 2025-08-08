# FoxESS + Home Assistant Solar Optimizer

👉 **Setup Guide:** see [`docs/setup.md`](docs/setup.md)

Automation pack for FoxESS + Home Assistant that:
- Calculates a **dynamic evening discharge start** to hit a target SoC floor by 23:30.
- Charges **off-peak** to ~90% by a set **deadline** with adaptive charge power.
- Adds **EV charging assist**: when the Tesla charges during peak (05:30–23:30), the battery force-charges from the grid (prevents dumping battery into the car).
- Includes a **self-healing watchdog** for off-peak mode and **ROI telemetry** (export revenue, import cost avoided, YTD gain, ROI%).

> **Sanitization:** Octopus entities use placeholders:  
> `octopus_energy_electricity_<MPAN>_<METER>_*`.  
> Replace with your own IDs from Developer Tools → States.

## Repo layout
home-assistant/
automations.yaml
configuration.snippet.yaml
scripts.yaml
lovelace.dashboard.yaml

## Install (HA)
1. Append `home-assistant/configuration.snippet.yaml` into your `configuration.yaml` (merge sections).
2. Replace/merge `automations.yaml` and `scripts.yaml`.
3. Create a dashboard from `lovelace.dashboard.yaml`.
4. Replace all `octopus_energy_electricity_<MPAN>_<METER>_*` with your real entities.  
5. Restart Home Assistant.

## Requirements
## Software
- Home Assistant 2024+
- HACS `foxess_modbus` (RS485 writes enabled)
- Octopus Energy integration (off-peak binary & import cost)
- Tesla integration (`binary_sensor.my_tesla_charging`) (or your equivalent)
## Hardware - based on my purchases (affiliated)
- Waveshare RS485 Modbus - https://amzn.to/4ftQm3T
- Home assistant green - https://amzn.to/3JoUd6l
- Decco WiFi Mesh - https://amzn.to/4lq4YCV

