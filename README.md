# 🐺 Wolf FGB Boiler Controller for Home Assistant

![Home Assistant](https://img.shields.io/badge/home%20assistant-%2341BDF5.svg?style=for-the-badge&logo=home-assistant&logoColor=white)
![MQTT](https://img.shields.io/badge/MQTT-660066?style=for-the-badge&logo=mqtt&logoColor=white)
![Wolf](https://img.shields.io/badge/Wolf-Heating-red?style=for-the-badge)
![License](https://img.shields.io/github/license/Ileriayo/markdown-badges?style=for-the-badge)

> **Unlock the full potential of your Wolf FGB-28 (and FGB Series) boiler via eBUS.** > No more read-only limitations. No more "3276.8" ghost values. Full Write Access & Clean Dashboard.

---

## 🧐 The Problem
Integrating Wolf FGB boilers with **ebusd** often results in:
1.  **Read-Only access:** You can see temperatures but can't change modes or curves.
2.  **Ghost Values:** Sensors reporting `3276.8 °C` or `32768` when in standby/idle.
3.  **Missing Entities:** Key parameters like Pump Mode (`Hg06`) or Anti-Cycle time (`Hg09`) are hidden because they lack units in the default configuration.

## 🚀 The Solution
This repository provides a complete configuration to:
* ✅ **Enable Write Access** to all `Hg` parameters (Hysteresis, Pump Power, Max Temp, etc.).
* ✅ **Fix Ghost Values** via Home Assistant Templating (converting `3276.8` to `Standby` or `Off`).
* ✅ **Expose Hidden Settings** by injecting a custom CSV definition.

---

## 🛠️ Installation & Setup

### 1. The "Magic" CSV (Wolf FGB)
The default ebusd configuration files are too strict. We modified the CSV to add Write (`w`) permissions and fix unit definitions, enabling automatic Slider discovery in Home Assistant.

**File:** `ebusd/08..hc.csv` (or your loaded csv)
> *Replace the content of your loaded CSV with the one provided in this repo.*

Key changes made:
- Added `r` (right) flag to `Hg01`-`Hg22` and fan speeds.
- Fixed units for Pump Power (`Hg16`, `Hg17`) to `%`.


### 2. ebusd Configuration
We use the **Official** ebusd MQTT configuration for stability, paired with the custom CSV.

**ebusd Add-on Configuration:**
```bash
scanconfig: false
loglevel_all: notice
mqtttopic: wolf_ebus
mqttint: /config/ebusd/wolf2026/mqtt-hassio.cfg
mqttjson: true
network_device: xxx.xxx.xxx.xxx:3333
accesslevel: "*"
mqttuser: mqtt
mqttpass: "123456"
mqtthost: xxx.xxx.xxx.xxx
mqttport: 1883
latency: 10
pollinterval: 5
configpath: /config/ebusd
