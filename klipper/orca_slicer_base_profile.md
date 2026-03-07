# Orca Slicer Base Profile (Klipper + RAMPS/Mega2560)

Use this as a manual setup template in Orca Slicer.
It is based on `klipper/config/printer.cfg`.

## 1) Printer Preset

- Printer technology: `FFF`
- G-code flavor: `Klipper`
- Bed type: `Rectangular`
- Bed size X: `200`
- Bed size Y: `170`
- Max print height Z: `200`
- Origin: `0,0` (front-left)
- Nozzle diameter: `0.4`
- Filament diameter: `1.75`

## 2) Motion Limits (Machine)

- Max velocity X: `300 mm/s`
- Max velocity Y: `300 mm/s`
- Max velocity Z: `5 mm/s`
- Max acceleration X: `1500 mm/s^2`
- Max acceleration Y: `1500 mm/s^2`
- Max acceleration Z: `100 mm/s^2`

## 3) Extruder/Temperature

- Hotend min temp: `0`
- Hotend max temp: `250`
- Bed max temp: `130`

## 4) Start G-code

Paste into Orca `Printer Settings -> Machine G-code -> Start G-code`:

```gcode
M140 S{first_layer_bed_temperature[0]}
M104 S{first_layer_temperature[0]}
G90
G28
BED_MESH_PROFILE LOAD=default
M190 S{first_layer_bed_temperature[0]}
M109 S{first_layer_temperature[0]}
G92 E0
G1 Z2.0 F3000
G1 X5 Y20 F6000
G1 Z0.28 F1200
G1 X5 Y150 E12 F1200
G1 X5.6 Y150 F6000
G1 X5.6 Y20 E24 F1200
G92 E0
```

## 5) End G-code

Paste into Orca `Printer Settings -> Machine G-code -> End G-code`:

```gcode
M400
G92 E0
G1 E-1.0 F1800
G91
G1 Z10 F600
G90
G1 X0 Y170 F6000
TURN_OFF_HEATERS
M107
```

## 6) Recommended Process Baseline

- Layer height: `0.20`
- First layer height: `0.24`
- Perimeters: `3`
- Top/Bottom layers: `5`
- Infill: `15%` (Grid/Gyroid)
- First layer speed: `20 mm/s`
- Outer wall speed: `40 mm/s`
- Inner wall speed: `60 mm/s`
- Infill speed: `80 mm/s`
- Travel speed: `150 mm/s`
- Print acceleration: `1000 mm/s^2`
- Travel acceleration: `1200 mm/s^2`

## 7) Retraction Baseline

Start here, then tune by print test:

- Retraction length: `2.0 mm`
- Retraction speed: `35 mm/s`
- Deretraction speed: `35 mm/s`
- Z hop: `0.2 mm`

## 8) Important Notes

- Your probe is configured with `x_offset=-5`, `y_offset=70` in Klipper.
- Mesh area is currently limited to `Y 110..130` in Klipper, so this profile loads an existing mesh (`BED_MESH_PROFILE LOAD=default`) instead of forcing a mesh calibration at each print.
- If Orca warns about placeholders, keep the same syntax shown above (Orca/Prusa style variables).
