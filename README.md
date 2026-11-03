# Deye / Sunsynk — Adaptive Grid Charge Current Blueprint (Home Assistant)

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fofedotov%2Fha-deye-adaptive-charge%2Fblob%2Fmain%2Fdeye_adaptive_grid_charge.yaml)

Home Assistant automation blueprint for **Deye / Sunsynk** hybrid inverters (connected locally via **Solarman**, **ESPHome**, or **Sunsynk Add-on**) that dynamically regulates **Max Grid Charge Current (`Modbus Reg 230`)** based on incoming AC grid voltage and total grid import power.

---

## Features & Control Algorithm (Asymmetric AIMD + Deadband)

1. **Instant Brownout Protection (`V_grid < V_critical`, default `< 193 V`):**
   Immediately drops `Reg 230` to `min_charge_current` (`0 A` or hardware min) to prevent grid undervoltage disconnects (`~185 V`) and off-grid overload trips.
2. **Proportional Step-Down (`V_critical <= V_grid < V_low` or `Grid Power > Max Grid Power`):**
   Reduces charge current proportionally (`3 A/V` below `200 V` by default, or `~1 A` per `58 W` above `max_grid_power_w`) with a `25 s` Modbus polling cooldown.
3. **Zero-Write Deadband (`200 V <= V_grid <= 208 V`):**
   Holds steady with **0 Modbus / EEPROM writes** once equilibrium is reached.
4. **Smooth Ramp-Up (`V_grid > 208 V`):**
   Steps up by `+5 A` every `150 s` up to `max_charge_current` (`50 A` default) while `SOC < 99%` and total grid power has headroom.
5. **Safe Post-Blackout & Full-Battery Parking (`V_grid < 150 V` or `SOC >= 99%`):**
   Parks `Reg 230` once at `safe_park_current` (`10 A` default) and freezes further EEPROM writes, ensuring a gentle soft-start when grid power returns after an outage.

---

## Required & Optional Entities

| Input | Deye Register | Example Entity (`Solarman` / `ESPHome`) | Required |
|---|---|---|---|
| **Grid Voltage** | `Reg 150` | `sensor.deye_grid_voltage` | **Yes** |
| **Grid Charge Current** | `Reg 230` | `number.deye_grid_charge_battery_current` | **Yes** |
| **Battery SOC %** | `Reg 184` | `sensor.deye_battery_soc` | Recommended |
| **Grid Power (W)** | `Reg 169` / `170` | `sensor.deye_total_grid_power` | Optional |

---

## Import & Update in Home Assistant

1. Go to **Settings → Automations & Scenes → Blueprints → Import Blueprint**.
2. Paste the URL of `deye_adaptive_grid_charge.yaml`:
   ```text
   https://github.com/ofedotov/ha-deye-adaptive-charge/blob/main/deye_adaptive_grid_charge.yaml
   ```
3. To pull future updates pushed to GitHub: open **Settings → Automations & Scenes → Blueprints**, click the **⋮** menu next to the blueprint, and click **Re-import blueprint**.
