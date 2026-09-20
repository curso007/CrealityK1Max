# Panda Breath Integration Guide  
## Creality K1 Max (Rooted Klipper)  

---

# Table of Contents

1. Introduction  
2. File Structure  
3. `panda_breath.cfg` — Core Configuration  
4. `panda_breath_macros.cfg` — Modes & Print Sequences  
5. `panda_breath_monitor.cfg` — Automatic Monitoring  
6. `panda_breath_dashboard.cfg` — Mainsail Menu  
7. `panda_breath_purge.cfg` — Smart Purge  
8. `panda_breath_filament.cfg` — Filament Profiles  
9. `panda_breath_dryer.cfg` — Filament Dryer  
10. Inclusion in `printer.cfg`  
11. README
12. ZIP Structure  
13. Notes & Safety  
14. Annexes  

---

# 1. Introduction

This manual provides a complete, modular, and professional configuration package for integrating the **Panda Breath environmental heater** with a **rooted Creality K1 Max** running full Klipper.

All configuration files are isolated, clean, and compatible with the Creality firmware structure.

---

# 2. File Structure

Place the following files inside:

```
/home/mks/klipper_config/includes/
```

Files included:

```
panda_breath.cfg
panda_breath_macros.cfg
panda_breath_monitor.cfg
panda_breath_dashboard.cfg
panda_breath_purge.cfg
panda_breath_filament.cfg
panda_breath_dryer.cfg
README.md
```

---

# 3. `panda_breath.cfg` — Core Configuration

```ini
[temperature_sensor panda_breath]
sensor_type: temperature_host
sensor_path: http://PandaBreath.local/api/temperature
min_temp: 0
max_temp: 80

[heater_generic panda_breath_heater]
heater_pin: host:panda_breath
sensor: panda_breath
control: watermark
max_power: 1.0
min_temp: 0
max_temp: 80

[verify_heater panda_breath_heater]
max_error: 120
check_gain_time: 60
hysteresis: 5
```

---

# 4. `panda_breath_macros.cfg` — Modes & Print Sequences

```ini
[gcode_macro PANDA_PERF]
gcode:
    {action_call_remote_method("panda_breath_set_mode", mode="performance")}

[gcode_macro PANDA_QUIET]
gcode:
    {action_call_remote_method("panda_breath_set_mode", mode="quiet")}

[gcode_macro PANDA_AUTO]
gcode:
    {action_call_remote_method("panda_breath_set_mode", mode="auto")}

[gcode_macro PANDA_PREHEAT]
gcode:
    PANDA_AUTO
    SET_HEATER_TEMPERATURE HEATER=panda_breath_heater TARGET=40
    M117 Panda Breath preheating...

[gcode_macro START_PRINT]
rename_existing: _START_PRINT
gcode:
    PANDA_PREHEAT
    _START_PRINT

[gcode_macro END_PRINT]
rename_existing: _END_PRINT
gcode:
    SET_HEATER_TEMPERATURE HEATER=panda_breath_heater TARGET=0
    PANDA_QUIET
    _END_PRINT
```

---

# 5. `panda_breath_monitor.cfg` — Automatic Monitoring

```ini
[gcode_macro PANDA_MONITOR]
gcode:
    {% set temp = printer["temperature_sensor panda_breath"].temperature %}
    {% if temp > 60 %}
        M117 Warning: Panda Breath > 60°C
    {% endif %}

[delayed_gcode PANDA_MONITOR_LOOP]
initial_duration: 10
gcode:
    PANDA_MONITOR
    UPDATE_DELAYED_GCODE ID=PANDA_MONITOR_LOOP DURATION=10
```

---

# 6. `panda_breath_dashboard.cfg` — Mainsail Menu

```ini
[menu __main __panda]
type: list
name: Panda Breath

[menu __main __panda __perf]
type: command
name: Mode Performance
gcode: PANDA_PERF

[menu __main __panda __quiet]
type: command
name: Mode Quiet
gcode: PANDA_QUIET

[menu __main __panda __auto]
type: command
name: Mode Auto
gcode: PANDA_AUTO

[menu __main __panda __test]
type: command
name: Test Heater
gcode: PANDA_TEST

[menu __main __panda __purge]
type: command
name: Purge Filament
gcode: PANDA_PURGE
```

---

# 7. `panda_breath_purge.cfg` — Smart Purge

```ini
[gcode_macro PANDA_PURGE]
gcode:
    {% set temp = printer["temperature_sensor panda_breath"].temperature %}
    {% if temp < 35 %}
        M117 Panda Breath not warm enough
        SET_HEATER_TEMPERATURE HEATER=panda_breath_heater TARGET=40
        G4 P5000
    {% endif %}
    M117 Purging...
    G1 E10 F300
    G1 E-2 F300
```

---

# 8. `panda_breath_filament.cfg` — Filament Profiles

```ini
[gcode_macro PANDA_PREHEAT_PLA]
gcode:
    PANDA_AUTO
    SET_HEATER_TEMPERATURE HEATER=panda_breath_heater TARGET=35
    M117 Preheat PLA...

[gcode_macro PANDA_PREHEAT_PETG]
gcode:
    PANDA_AUTO
    SET_HEATER_TEMPERATURE HEATER=panda_breath_heater TARGET=45
    M117 Preheat PETG...

[gcode_macro PANDA_PREHEAT_ABS]
gcode:
    PANDA_PERF
    SET_HEATER_TEMPERATURE HEATER=panda_breath_heater TARGET=55
    M117 Preheat ABS...

[gcode_macro PANDA_PREHEAT_NYLON]
gcode:
    PANDA_PERF
    SET_HEATER_TEMPERATURE HEATER=panda_breath_heater TARGET=60
    M117 Preheat Nylon...
```

---

# 9. `panda_breath_dryer.cfg` — Filament Dryer

```ini
[gcode_macro PANDA_DRY]
gcode:
    {% set target = params.TARGET|default(50)|int %}
    PANDA_PERF
    SET_HEATER_TEMPERATURE HEATER=panda_breath_heater TARGET={target}
    M117 Drying filament...
    G4 P3600000

[gcode_macro PANDA_DRY_2H]
gcode:
    PANDA_PERF
    SET_HEATER_TEMPERATURE HEATER=panda_breath_heater TARGET=55
    M117 Drying 2h...
    G4 P7200000
```

---

# 10. Inclusion in `printer.cfg`

```ini
[include includes/panda_breath.cfg]
[include includes/panda_breath_macros.cfg]
[include includes/panda_breath_monitor.cfg]
[include includes/panda_breath_dashboard.cfg]
[include includes/panda_breath_purge.cfg]
[include includes/panda_breath_filament.cfg]
[include includes/panda_breath_dryer.cfg]
```

---

# 11. README

```md
# Panda Breath Integration — Creality K1 Max (Rooted)

Place all .cfg files inside:
`/home/mks/klipper_config/includes/`

Include them in `printer.cfg` as documented.
```

---

# 12. ZIP Structure

```
panda_breath/
│
├── includes/
│   ├── panda_breath.cfg
│   ├── panda_breath_macros.cfg
│   ├── panda_breath_monitor.cfg
│   ├── panda_breath_dashboard.cfg
│   ├── panda_breath_purge.cfg
│   ├── panda_breath_filament.cfg
│   └── panda_breath_dryer.cfg
│
└── README.md
```

---

# 13. Notes & Safety

- Do not exceed 80°C on Panda Breath.  
- Avoid running the dryer during printing.  
- Ensure proper ventilation.  

---

# 14. Annexes
[Configuration zip](./panda_breath.zip)