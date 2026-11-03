# Deye / Sunsynk — Adaptive Grid Charge Current Blueprint (Home Assistant)

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fofedotov%2Fha-deye-adaptive-charge%2Fblob%2Fmain%2Fdeye_adaptive_grid_charge.yaml)

Home Assistant automation blueprint for **Deye / Sunsynk** hybrid inverters (connected locally via **Solarman**, **ESPHome**, or **Sunsynk Add-on**) that dynamically regulates **Max Grid Charge Current (`Modbus Reg 230`)** relative to the inverter's **Grid Low-Voltage Disconnect threshold (`V_cutoff`)** and total grid import power.

---

## Automatic Thresholds from Inverter Cutoff (`V_cutoff`)

Provide either the inverter's **`Grid Voltage Low` entity (`Reg 288`)** or set **`V_cutoff` manually** (default `185 V` for EU `EN50549` IP Protection). All operating zones are calculated automatically from `V_cutoff`:

* **`V_critical` = `V_cutoff + 8 V`** *(e.g. `193 V`)* → Instant drop to `min_charge_current` (`0 A`)
* **`V_low` = `V_cutoff + 15 V`** *(e.g. `200 V`)* → Proportional step-down (`3 A/V` below `V_low` or when `Grid Power > Max Grid Power`)
* **Deadband = `V_low .. V_high`** *(e.g. `200..208 V`, `+8 V` width)* → **0 Modbus / EEPROM writes**
* **Ramp-Up = `> V_high`** *(e.g. `> 208 V`)* → Smooth `+5 A` every `150 s` up to `max_charge_current` (`50 A`)
* **Off-Grid / Full-SoC Parking (`V_grid < 100 V`, `Grid Connected = off`, or `SOC >= 99%`)** → Parks `Reg 230` once at `safe_park_current` (`10 A`) for soft post-outage reconnects and zero idle EEPROM wear.

---

## Required & Optional Entities

| Input | Deye Register | Example Entity (`Solarman` / `ESPHome`) | Required |
|---|---|---|---|
| **Grid Voltage** | `Reg 150` | `sensor.inverter_grid_voltage` | **Yes** |
| **Grid Charge Current** | `Reg 230` | `number.inverter_grid_charge_battery_current` | **Yes** |
| **Inverter Grid Voltage Low** | `Reg 288` | `number.inverter_grid_voltage_low` *(or leave empty to use slider)* | Optional |
| **Grid Connected Status** | `Reg 194` | `binary_sensor.inverter_grid_connected_status` | Optional |
| **Battery SOC %** | `Reg 184` | `sensor.inverter_battery_soc` | Recommended |
| **Grid Power (W)** | `Reg 169` / `170` | `sensor.inverter_grid_power` | Optional |

---

## Import & Update in Home Assistant

1. Go to **Settings → Automations & Scenes → Blueprints → Import Blueprint** and paste:
   ```text
   https://github.com/ofedotov/ha-deye-adaptive-charge/blob/main/deye_adaptive_grid_charge.yaml
   ```
2. To update later: open **Settings → Automations & Scenes → Blueprints**, click **⋮** next to the blueprint, and click **Re-import blueprint**.
