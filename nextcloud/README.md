Nextcloud + HDD setup

What was detected
- Your system disk: /dev/sda (mounted as /)
- Your external HDD: /dev/sdb1 (label: HDD1TB, filesystem: ntfs)
- Current state: /dev/sdb1 is not mounted

1) Install NTFS support (one-time)

sudo apt update
sudo apt install -y ntfs-3g

2) Mount HDD at a stable path

sudo mkdir -p /mnt/hdd1tb
sudo mount -t ntfs3 -o uid=1000,gid=1000,umask=022 /dev/sdb1 /mnt/hdd1tb

If ntfs3 fails, use:
sudo mount -t ntfs-3g -o uid=1000,gid=1000,umask=022 /dev/sdb1 /mnt/hdd1tb

3) Create app data folder on HDD

mkdir -p /mnt/hdd1tb/nextcloud

4) Configure environment

cd /home/andres/docker-app-pi/nextcloud
cp .env.example .env

Edit .env and set:
- MYSQL_ROOT_PASSWORD
- MYSQL_PASSWORD
- NEXTCLOUD_TRUSTED_DOMAINS (server IP/domain)

5) Start Nextcloud stack

docker compose up -d

6) Open Nextcloud

http://SERVER_IP:8088

At first load, create admin user and keep data folder as:
/var/www/html/data

Important
- This stack stores all Nextcloud files (app + data) on the HDD path from NEXTCLOUD_DATA_PATH.
- For better performance and backup flexibility, you may later split data and app config into separate mounts.

Optional: auto-mount on reboot

1. Get UUID:
   sudo blkid /dev/sdb1

2. Add to /etc/fstab (replace UUID value):
   UUID=YOUR_UUID /mnt/hdd1tb ntfs3 defaults,uid=1000,gid=1000,umask=022,nofail 0 0

3. Test fstab:
   sudo mount -a