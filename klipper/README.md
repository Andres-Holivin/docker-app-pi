# Klipper on Ubuntu Server (Simple)

This folder runs:
- Klipper
- Moonraker
- Mainsail
- Webcam (mjpg-streamer)

## 1) Start everything

```bash
cd ~/docker-app-pi/klipper
mkdir -p config run gcodes logs
sudo chown -R $USER:$USER run gcodes logs
chmod 775 run gcodes logs
docker compose up -d
docker compose ps
```

Open Mainsail:

```text
http://<your-server-ip>:8080
```

## 2) Webcam

Webcam is exposed on:

```text
http://<your-server-ip>:8081/?action=stream
```

It is already registered in Moonraker, so Mainsail should show it automatically.

If camera is black, switch from `/dev/video0` to `/dev/video1` in `docker-compose.yml`, then restart:

```bash
cd ~/docker-app-pi/klipper
docker compose up -d webcam moonraker
```

## 3) If MCU is not connecting (`Unable to connect` / `Serial connection closed`)

Run this (it auto-builds and flashes Klipper firmware for Mega2560/RAMPS):

```bash
cd ~/docker-app-pi/klipper
docker compose stop klipper moonraker
docker compose run --rm --user root --entrypoint sh klipper -lc 'set -e; apt-get update >/dev/null; DEBIAN_FRONTEND=noninteractive apt-get install -y make gcc-avr avr-libc avrdude >/dev/null; cd /opt/klipper; make olddefconfig; make clean >/dev/null; make -j$(nproc); make flash FLASH_DEVICE=/dev/serial/by-id/usb-1a86_USB_Serial-if00-port0'
docker compose up -d klipper moonraker
docker compose logs --tail=80 klipper
```

Expected in logs:
- `Loaded MCU 'mcu' ...`
- `Configured MCU 'mcu' ...`

## 4) If USB ID changed

```bash
ls -l /dev/serial/by-id/
```

Then update `config/printer.cfg`:

```ini
[mcu]
serial: /dev/serial/by-id/<your-device>
```

Restart:

```bash
docker compose restart klipper moonraker
```

## 5) Servo Z Probe (auto bed leveling)

`printer.cfg` now includes:
- probe-based Z homing
- servo probe on RAMPS `D11`
- `safe_z_home`
- `bed_mesh`

Important: the current probe offset is `x_offset: 70`, `y_offset: -4`.
That means all probe-based operations must use XY positions chosen for the
probe, not just the nozzle. The current config homes and calibrates at nozzle
position `X40 Y59`, which places the probe at `X110 Y55`. The current mesh is
kept inside probe coordinates `35,15` to `185,95` with `horizontal_move_z: 10`
to avoid starting a probe move too close to the bed.

Important: in this config the X endstop is not `X0`. The X home switch is
`X=-40`, while `X0 Y0` is the front-left bed origin used for printing.

First-time checks in Mainsail console:

```text
QUERY_PROBE
SET_SERVO SERVO=z_probe_servo ANGLE=25
SET_SERVO SERVO=z_probe_servo ANGLE=180
G28
CALIBRATE_Z_OFFSET
SAVE_CONFIG
BED_MESH_CALIBRATE
SAVE_CONFIG
```

If you want to run it manually without the macro, do this exact sequence:

```text
G28
G1 X40 Y59 F6000
PROBE_CALIBRATE
```

After `SAVE_CONFIG`, Klipper restarts and forgets the current position. If you
want it parked at `X0 Y0` after the restart, run:

```text
PARK_FRONT_LEFT
```

Do not start `PROBE_CALIBRATE` from an arbitrary XY position. With the current
probe offset, mesh and calibration coordinates must stay inside the probe's
reachable bed area or Klipper will halt with `Move out of range`.

If deploy/stow is reversed, swap the two servo angles in `config/printer.cfg`.

## 6) Slicer end G-code

To finish every print at `X0 Y0`, set Orca `Printer Settings -> Machine G-code
-> End G-code` to:

```gcode
END_PRINT
```

`END_PRINT` is defined in `config/printer.cfg` and already does:
- retract
- lift Z
- move to `X0 Y0`
- turn heaters and fan off

If you want the nozzle to go to the physical X/Y switches instead of the bed
origin, that is `G28 X Y`, not `X0 Y0`.

Important: this only affects newly sliced files. If a `.gcode` file was already
exported, it keeps the old end G-code until you re-slice it.

## 7) Daily commands

```bash
cd ~/docker-app-pi/klipper
docker compose ps
docker compose logs -f klipper
docker compose restart klipper moonraker
docker compose pull && docker compose up -d
```

## 8) Useful files

- `config/printer.cfg`
- `config/moonraker.conf`
- `config/mainsail.json`
- `gcodes/`
- `logs/`
