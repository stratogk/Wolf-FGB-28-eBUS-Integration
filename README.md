# 🐺 Wolf FGB Unlocked: Ultimate Home Assistant Control

![Home Assistant](https://img.shields.io/badge/home%20assistant-%2341BDF5.svg?style=for-the-badge&logo=home-assistant&logoColor=white)
![MQTT](https://img.shields.io/badge/MQTT-660066?style=for-the-badge&logo=mqtt&logoColor=white)
![Wolf](https://img.shields.io/badge/Wolf-Heating-red?style=for-the-badge)
![License](https://img.shields.io/github/license/Ileriayo/markdown-badges?style=for-the-badge)

> **Unlock the full potential of your Wolf FGB-28 (and FGB Series) boiler via eBUS.** > No more read-only limitations. No more "3276.8" ghost values. Full Write Access & Clean Dashboard.

---

## 🧐 The Problem
Integrating Wolf FGB boilers with **ebusd** often results in frustating limitations:
1.  **Read-Only Access:** Default configs allow you to see temperatures but NOT change modes or curves.
2.  **Ghost Values:** Sensors reporting `3276.8 °C` or `32768` (standby/no-sensor code) instead of `Off` or `Standby`.
3.  **Missing Entities:** Key parameters like Pump Mode (`Hg06`) or Anti-Cycle time (`Hg09`) are hidden because they lack units in the default CSV.

## 🚀 The Solution
This repository provides a complete configuration to:
* ✅ **Enable Write Access** to all `Hg` parameters (Hysteresis, Pump Power, Max Temp, etc.) by patching the CSV.
* ✅ **Fix Ghost Values** via Home Assistant Templating.
* ✅ **Expose Hidden Settings** and create manual Sliders for full control.

---

## 🛠️ Installation & Setup

### Step 1: The "Hacked" CSV
The default ebusd configuration files are too strict. We modified the CSV to add Write (`w`) flags and fix unit definitions, enabling automatic Slider discovery or manual control.

**File:** `ebusd/15.700.csv` (or your specific boiler definition)
> *Replace the content of your loaded CSV with the modified version provided in this repo.*

**Key changes:**
- Added `w` (write) flag to `Hg01`-`Hg22` and fan speeds.
- Fixed units for Pump Power (`Hg16`, `Hg17`) to `%`.
- Added units to time-based parameters to enable sliders.

### Step 2: ebusd Configuration
We use the **Official** ebusd MQTT configuration for stability (to prevent read errors), paired with our custom CSV.

**ebusd Add-on Configuration:**

--scanconfig --mqttport=1883 --mqttjson --mqttint=/etc/ebusd/mqtt-hassio.cfg


Step 3: Home Assistant Configuration (configuration.yaml)
Add the following to your configuration.yaml to clean up the data and create manual controls (since the official config might block write access):

YAML

# 1. CLEAN UP GHOST VALUES (3276.8 -> Off/Standby)

template:

  - sensor:
  - 
      - name: "Wolf Flow Temp Desired Fixed"
        unique_id: "wolf_flow_temp_desired_fixed"
        state: >
          {% set val = states('sensor.ebusd_hc_flowtempdesired') | float(0) %}
          {% if val > 100 %} Standby {% else %} {{ val }} {% endif %}
        unit_of_measurement: "°C"
        icon: mdi:target
      - name: "Wolf Pump Status Fixed"
        unique_id: "wolf_pump_status_fixed"
        state: >
          {% set val = states('sensor.ebusd_hc_pump') | float(0) %}
          {% if val > 100 %} 0 {% else %} {{ val }} {% endif %}
        unit_of_measurement: "%"
        icon: mdi:pump
      - name: "Wolf Max Boiler Temp Hg22 Fixed"
        unique_id: "wolf_hg22_fixed"
        state: >
          {% set val = states('sensor.ebusd_hc_hg22') | float(0) %}
          {% if val > 150 %} Off {% else %} {{ val }} {% endif %}
        unit_of_measurement: "°C"
        icon: mdi:thermometer-alert

# 2. MANUAL CONTROLS (Sliders for Settings)

mqtt:

  number:
  
    - name: "Wolf Target Temp Control"
      unique_id: "wolf_target_temp_control"
      command_topic: "ebusd/hc/Flowtempdesired/set"
      state_topic: "ebusd/hc/Flowtempdesired"
      min: 20
      max: 80
      step: 1
      unit_of_measurement: "°C"
      mode: slider
    - name: "Wolf Max Heating Power Hg03"
      unique_id: "wolf_max_power_hg03"
      command_topic: "ebusd/hc/Hg03/set"
      state_topic: "ebusd/hc/Hg03"
      min: 0
      max: 100
      step: 1
      unit_of_measurement: "%"
      icon: mdi:fire
    
 # Add other Hg parameters here following the same pattern...




📊 Pro Dashboard (Lovelace)
A clean, professional interface to monitor and control your boiler.

Copy this into a "Manual" card:

YAML

type: vertical-stack
title: Wolf FGB Professional Monitor
cards:
  # --- LIVE MONITORING ---
  - type: grid
    columns: 2
    square: false
    cards:
      - type: gauge
        entity: sensor.ebusd_hc_flowtemp
        name: Flow Temp
        min: 0
        max: 90
        severity:
          green: 30
          yellow: 60
          red: 75
      - type: gauge
        entity: sensor.wolf_pump_status_fixed
        name: Pump Power
        unit: '%'
        min: 0
        max: 100
        severity:
          green: 1
          yellow: 50
          red: 90

  # --- CONTROLS ---
  
  - type: entities
    title: 🔥 Control Panel
    show_header_toggle: false
    entities:
      - entity: number.wolf_target_temp_control
        name: Target Water Temp
      - entity: number.wolf_max_heating_power_hg03
        name: Max Power (Hg03)
      - entity: sensor.wolf_max_boiler_temp_hg22_fixed
        name: Max Limit (Hg22)

        
⚠️ Disclaimer
Use at your own risk. Writing values to your boiler (e.g., changing max temperatures or pump speeds) can affect its operation. Always verify Hg parameters with your boiler's official manual.


⭐ Credits
ebusd: The core engine making this possible.

Home Assistant: The automation platform.
