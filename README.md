# FoxESS + Home Assistant Solar Optimizer
Fairly new to it all (use at own risk)

👉 **Setup Guide:** see [`docs/setup.md`](docs/setup.md)

Automation pack for a **FoxESS EP11** battery/inverter + **Octopus (Intelligent Octopus Go)** on Home Assistant. It runs the battery as a price-arbitrage + self-consumption asset and keeps the inverter under one consistent controller:

- **Dynamic evening discharge** — calculates a discharge start time so the battery lands on a target SoC floor by 23:30, exporting at the peak rate.
- **Adaptive off-peak charging** — on the cheap window it switches to Force Charge and recomputes charge power every minute to hit ~max-SoC by a user-set deadline, with a **self-healing watchdog** and a post-restart **recover** step.
- **Tesla peak-assist** — when the car charges during peak (05:30–23:30), the battery force-charges from the grid so it isn't dumped into the car; heartbeat + restart-enforce keep it sticky.
- **Octopus perk integration** — auto force-charge during **Greener Nights** and **Octoplus Free Electricity** sessions, and **auto-join Saving Sessions**.
- **Safety watchdogs** — alert on unwanted solar→battery charging (when export > charge value) and on the inverter getting stuck in Force Charge.
- **ROI telemetry** — export revenue, import cost avoided, battery arbitrage profit, daily net profit.

## Sanitization
All Octopus identifiers are **placeholders** — replace with your own from **Developer Tools → States**. Don't publish your real IDs.

| Placeholder | What it is |
|---|---|
| `<METER>` | electricity meter serial |
| `<MPAN>` | import MPAN |
| `<MPAN_EXPORT>` | export MPAN |
| `a_<ACCOUNT>` | Octopus account id (used in calendar/perk entities) |
| `<INTELLIGENT_DEVICE>` | Intelligent Octopus dispatching device id |

## Repo layout
```
home-assistant/
  configuration.snippet.yaml   # helpers, template sensors, statistics, utility meters
  automations.yaml             # the optimizer (24 automations)
  scripts.yaml                 # reusable inverter mode scripts
  dashboards/
    energy_ops.yaml            # "Energy Ops" sections dashboard
docs/
  setup.md                     # full wiring + install guide
```

## Install (HA)
1. Merge `home-assistant/configuration.snippet.yaml` into your `configuration.yaml`.
2. Merge `home-assistant/automations.yaml` and `home-assistant/scripts.yaml`.
3. Add `home-assistant/dashboards/energy_ops.yaml` as a YAML dashboard.
4. Replace every placeholder (`<METER>`, `<MPAN>`, `a_<ACCOUNT>`, …) with your real Octopus IDs.
5. Restart Home Assistant.

See [`docs/setup.md`](docs/setup.md) for the RS485 wiring, required HACS cards, and a how-it-works walkthrough.

## Requirements
### Software
- Home Assistant 2024.6+ (for the Sections dashboard)
- HACS `foxess_modbus` (RS485 writes enabled)
- Octopus Energy integration (off-peak binary, import/export cost, perk calendars)
- Tesla integration (`binary_sensor.my_tesla_charging`) or your own EV-charging flag

### Hardware — based on my purchases (affiliated)
- Waveshare RS485 Modbus - https://amzn.to/4ftQm3T
- Home assistant green - https://amzn.to/3JoUd6l
- Decco WiFi Mesh - https://amzn.to/4lq4YCV
