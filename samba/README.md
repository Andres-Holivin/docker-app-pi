# Samba Docker

## 1) Verify external HDD is mounted on host

This project expects the HDD at:

`/mnt/hdd1tb`

Check it with:

```bash
lsblk -f
mount | grep /mnt/hdd1tb
```

## 2) Configure

```bash
cd samba
cp .env.example .env
# edit .env and change SAMBA_PASS
```

## 3) Start

```bash
docker compose up -d
```

## 4) Access from network

Use your server IP and share name:

- `\\SERVER_IP\hdd` (Windows)
- `smb://SERVER_IP/hdd` (macOS/Linux)

Login with `SAMBA_USER` and `SAMBA_PASS`.

## Notes

- If your HDD mount path changes, update `SAMBA_HDD_PATH` in `.env`.
- This setup publishes SMB ports 137/138/139/445 on the host.
