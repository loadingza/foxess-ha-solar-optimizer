# Setup Guide

This pack optimises a FoxESS + Home Assistant system for:
- **Evening export**: dynamic start time to reach a target SoC floor by **23:30**.
- **Off-peak charging**: adaptive power to hit ~**90% SoC** by a user-set **deadline**.
- **EV charging assist**: during **05:30–23:30**, force-charge the battery from the grid when your EV starts charging (prevents dumping battery into the car).
- **Watchdog**: self-heals off-peak charge mode if it drifts.
- **ROI telemetry**: export revenue, import cost avoided, YTD gain, ROI%.

> **Sanitisation:** Octopus entities use placeholders  
> `octopus_energy_electricity_<MPAN>_<METER>_*`  
> Replace with your IDs from **Developer Tools → States**. Don’t publish your real IDs.

---

## Step 0 — Install RS485 Modbus control (FoxESS writes)

This project assumes **control via RS485 (Modbus RTU)** using the HACS `foxess_modbus` integration.  
Telemetry can be via LAN/Cloud; RS485 is used for reliable **write** access (work mode, charge/discharge power).

**Hardware used (links):**
- Waveshare Industrial USB-to-RS485 (CH343G): https://amzn.to/4ftQm3T  
- Home Assistant Green: https://amzn.to/3JoUd6l  
- Deco WiFi Mesh: https://amzn.to/4lq4YCV

**Wiring reference:**  
Follow the excellent guide here (A/B polarity, termination, grounding):  
https://github.com/nathanmarlor/foxess_modbus/wiki/Modbus-Wiring-Guide

**Quick steps:**
1. **Wire RS485** from inverter to adapter (A↔A, B↔B). Fit/enable termination where appropriate per the guide above.
2. **Plug adapter** into HA Green. Confirm the serial device path (Supervisor → System → *Hardware*), e.g. `/dev/ttyUSB0`.
3. **Install HACS** (if not already), then **install “FoxESS Modbus”** from HACS.
4. **Add Integration** → *FoxESS Modbus* → choose **Serial/RTU** → set:
   - **Serial port**: `/dev/ttyUSB0` (or your path)
   - **Baud/parity/stop bits**: use the integration defaults / your inverter label
   - Enable **write registers** / control entities as per integration docs
5. Verify the following entities exist (names may vary):  
   `select.work_mode`, `number.force_charge_power`, `number.force_discharge_power`,  
   `sensor.battery_soc`, `sensor.load_power`, `sensor.pv_power`, `sensor.invbatpower`, `sensor.rpower`,  
   `sensor.pv1_power`, `sensor.pv2_power`, `sensor.bms_cell_temp_high`, `sensor.invtemp`, and either  
   `sensor.net_grid_power` or `sensor.grid_ct`.

> If you already use LAN/Modbus TCP for fast **telemetry**, that’s fine. Keep **RS485** for **control**.

---

## Requirements

### Home Assistant
- **2024.6+** recommended (for the **Sections** view).
- Profile → **Enable Advanced Mode**.

### Integrations
- **FoxESS** via HACS `foxess_modbus` (RS485 writes) or FoxESS Cloud (read only).
- **Octopus Energy**:  
  `binary_sensor.octopus_energy_electricity_<MPAN>_<METER>_off_peak`  
  `sensor.octopus_energy_electricity_<MPAN>_<METER>_current_accumulative_cost`
- **EV charging flag** (any binary sensor). Default in this pack:  
  `binary_sensor.my_tesla_charging`
- **Weather**: `weather.forecast_home` (Met.no).

### Custom Lovelace Cards (HACS)
- **ApexCharts Card** (RomRider)  
  Resource: `/hacsfiles/apexcharts-card/apexcharts-card.js` (type: `module`)
- **Gauge Card Pro** (optional)  
  Resource: `/hacsfiles/gauge-card-pro/gauge-card-pro.js` (type: `module`)  
  > You can swap to built-in `gauge` if you prefer.

---

## Install

### 1) Add the config snippet
Open **`configuration.yaml`** and make sure you include:
```yaml
default_config:
frontend:
  themes: !include_dir_merge_named themes
automation: !include automations.yaml
script: !include scripts.yaml
scene: !include scenes.yaml

Append/merge home-assistant/configuration.snippet.yaml into your configuration.yaml.
This defines:
	•	Helpers: input_boolean.fox_remote_control_enabled, input_number.*, input_datetime.charge_deadline
	•	Template sensors (incl. *_kw power views, planning maths, ROI)
	•	Statistics sensors (evening/overnight averages)
	•	Utility meters (total_financial_gain_ytd, daily charge/discharge)

2) Automations & scripts
	•	Replace/merge automations.yaml with home-assistant/automations.yaml.
	•	Replace/merge scripts.yaml with home-assistant/scripts.yaml.

3) Dashboard
	•	Install the cards in HACS → Frontend.
	•	Check resources (Settings → Dashboards → Resources):
add the two URLs above if HACS didn’t auto-add them.
	•	Create a dashboard (Settings → Dashboards → + Add Dashboard → “Solar & Battery”).
	•	Open it → ⋮ → Edit dashboard → Raw configuration editor → paste home-assistant/lovelace.dashboard.yaml.

4) Replace Octopus placeholders

Search for:
	•	binary_sensor.octopus_energy_electricity_<MPAN>_<METER>_off_peak
	•	sensor.octopus_energy_electricity_<MPAN>_<METER>_current_accumulative_cost

Replace <MPAN> and <METER> with your exact IDs from Developer Tools → States.

5) Restart Home Assistant

Settings → System → Restart. After boot, give it ~60s for templates/stats to populate.

⸻

How it works (quick)
	•	Daytime default → 05:30 sets Feed-in First.
	•	Evening export → starts at the calculated time so you reach the SoC floor by 23:30; uses Force Discharge at your target kW unless house load already exceeds that (then leaves Feed-in First).
	•	Off-peak → on off-peak start, switches to Force Charge and adjusts charge power each minute to meet the deadline (~90% SoC).
	•	EV assist → when binary_sensor.my_tesla_charging turns on between 05:30–23:30, sets Force Charge at 2.5 kW (from grid). When charging ends or at 23:30, runs set_feed_in_first_mode.
	•	Watchdog → every 5 minutes during off-peak, if not full and not in Force Charge, it nudges back and notifies.

⸻

Entity checklist

Helpers: input_boolean.fox_remote_control_enabled, input_number.target_discharge_rate_kw, input_number.battery_discharge_soc_target, input_datetime.charge_deadline
Planning: sensor.planned_average_charge_rate, sensor.calculated_dynamic_discharge_start_time, sensor.required_dynamic_charge_power
kW views: sensor.pv_power_kw, sensor.load_power_kw, sensor.invbatpower_kw, sensor.rpower_kw, sensor.pv1_power_kw, sensor.pv2_power_kw, sensor.net_grid_power_kw
Battery/temps: sensor.battery_soc, sensor.battery_soh, sensor.bms_cell_temp_high, sensor.invtemp
Money/ROI: sensor.daily_export_revenue, sensor.daily_import_cost_avoided, sensor.total_daily_financial_gain, sensor.daily_net_profit, sensor.total_financial_gain_ytd, sensor.current_roi_percentage
Octopus: binary_sensor.octopus_energy_electricity_<MPAN>_<METER>_off_peak, sensor.octopus_energy_electricity_<MPAN>_<METER>_current_accumulative_cost
EV flag: binary_sensor.my_tesla_charging

⸻

Tuning
	•	Target discharge rate: input_number.target_discharge_rate_kw (default 2.5 kW).
	•	Evening SoC floor: input_number.battery_discharge_soc_target (default 20%).
	•	Hard SoC floor: input_number.battery_hard_soc_floor (default 10%).
	•	Charge deadline: input_datetime.charge_deadline (default 05:20).
	•	Investment (ROI): input_number.initial_solar_investment (default £12,800).

Money sensors currently use £0.287/kWh (peak) and £0.15/kWh (export).
If your rates differ, edit those two templates in configuration.snippet.yaml (or convert to input_numbers).

⸻

Troubleshooting
	•	Card type missing / red errors → add the /hacsfiles/... resources and hard-refresh.
	•	Unknown entity → merge the snippet and adjust entity names to yours.
	•	Units off → dashboard must reference the *_kw sensors.
	•	Off-peak didn’t start → off-peak binary must be on; ensure fox_remote_control_enabled is on.
	•	EV assist didn’t trigger → confirm binary_sensor.my_tesla_charging toggles on and you’re within 05:30–23:30.
	•	Competing automations → keep fox_remote_control_enabled off until you’re ready to hand control to this pack.

⸻

Safety & warranty
	•	Power setpoints assume kW. If your number.force_*_power requires Watts, multiply values by 1000 in the service calls.
	•	Battery cycling contributes to throughput; tune SoC targets/export rate to your warranty tolerance.
