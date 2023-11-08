# Docknode pol

## Endpoints

- Fullnode: `https://mydomain.com`
- Node exporter: `https://mydomain.com:9100/metrics`

## Install

0. VPS config (optional)

```bash
apt update
apt upgrade
apt install git
apt install screen

# Or all in one command
apt update && apt upgrade -y && apt install -y git screen

# install docker https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository


# Prepare disks (8 To)
# Example: merge sda (4To) and sdb (4To)
apt install lvm2

parted /dev/sda
# mklabel gpt
# mkpart primary 0% 100%
# quit

parted /dev/sdb
# mklabel gpt
# mkpart primary 0% 100%
# quit

pvcreate /dev/sda1
pvcreate /dev/sdb1
vgcreate vg0 /dev/sda1 /dev/sdb1
lvcreate -l 100%FREE -n media vg0
mke2fs -t ext4 /dev/vg0/media


nano /etc/fstab
# Add this line at the end of the file:
# /dev/vg0/media /mnt/media ext4 defaults 0 1

reboot
cd /mnt/media
```

1. Clone the repository and

```bash
git clone https://github.com/olivbau/docknode-pol.git
cd docknode-pol
```

2. Create config files

```bash
docker run -v ./heimdall/config:/heimdall-home/config:rw -v ./heimdall/data:/heimdall-home/data:rw 0xpolygon/heimdall:latest init --home=/heimdall-home
```

3. Configure environement variables

```bash
cp .env.example .env

# Generate users passwords for basic auth
docker run --rm caddy:2-alpine caddy hash-password --plaintext 'password'

# Set users and passwords for basic auth
# Set the host
nano .env
```

4. Setup UFW

```bash
ufw allow ssh
ufw allow 26656
ufw deny 26657
ufw deny 1317
ufw enable
```

5. Run

```bash
docker compose pull


docker compose up -d
docker logs -f docknode-pol-heimdall-1 --since 5m
docker compose down
```

```bash
curl -L https://snapshot-download.polygon.technology/snapdown.sh | bash -s -- --network mainnet --client heimdall --extract-dir ./heimdall/data --validate-checksum false
curl -L https://snapshot-download.polygon.technology/snapdown.sh | bash -s -- --network mainnet --client bor --extract-dir ./bor/data --validate-checksum false
```
