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

First-time checks in Mainsail console:

```text
QUERY_PROBE
SET_SERVO SERVO=z_probe_servo ANGLE=10
SET_SERVO SERVO=z_probe_servo ANGLE=90
G28
PROBE_CALIBRATE
SAVE_CONFIG
BED_MESH_CALIBRATE
SAVE_CONFIG
```

If deploy/stow is reversed, swap the two servo angles in `config/printer.cfg`.

## 6) Daily commands

```bash
cd ~/docker-app-pi/klipper
docker compose ps
docker compose logs -f klipper
docker compose restart klipper moonraker
docker compose pull && docker compose up -d
```

## 7) Useful files

- `config/printer.cfg`
- `config/moonraker.conf`
- `config/mainsail.json`
- `gcodes/`
- `logs/`
