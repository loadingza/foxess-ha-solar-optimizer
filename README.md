# FoxESS + Home Assistant Solar Optimizer

Automation pack for FoxESS + Home Assistant that:
- Calculates a **dynamic evening discharge start** to hit a target SoC floor by 23:30.
- Charges **off-peak** to ~90% by a set **deadline** with adaptive charge power.
- Adds **EV charging assist**: when the Tesla charges during peak (05:30–23:30), the battery force-charges from the grid (prevents dumping battery into the car).
- Includes a **self-healing watchdog** for off-peak mode and **ROI telemetry** (export revenue, import cost avoided, YTD gain, ROI%).

> **Sanitization:** Octopus entities use placeholders:  
> `octopus_energy_electricity_<MPAN>_<METER>_*`.  
> Replace with your own IDs from Developer Tools → States.


