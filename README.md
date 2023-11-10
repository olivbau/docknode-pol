# Docknode pol

## Endpoints

- Fullnode: `https://mydomain.com`
- Node exporter: `https://mydomain.com:9100/metrics`

## Prerequisites

```bash
# Install git and screen
apt update && apt upgrade -y && apt install -y git screen

# Install docker
# https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository
```

## How to use

0. Disks config (optional)

```bash
# How to create a 8TB disk from two 4TB disks
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

1. Clone the repository

```bash
git clone https://github.com/olivbau/docknode-pol.git
cd docknode-pol
```

2. Generate config files

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

4. Snapshots (optional)

```bash
# Using screen is recommended
# Create screen session (screen -S snapshots)
# Leave screen session (CTRL+A+D)
# Attache screen session (screen -R snapshots)
# Kill screen sessuin (screen -S snapshots -X quit)

curl -L https://snapshot-download.polygon.technology/snapdown.sh | bash -s -- --network mainnet --client heimdall --extract-dir ./heimdall/data --validate-checksum true
curl -L https://snapshot-download.polygon.technology/snapdown.sh | bash -s -- --network mainnet --client bor --extract-dir ./bor/chaindata --validate-checksum true
```

5. Setup UFW

```bash
ufw allow ssh
ufw allow 26656
ufw deny 26657
ufw deny 1317
ufw deny 8545
ufw enable
```

6. Run

```bash
docker compose pull
docker compose up -d
```

## Useful commands

```bash
docker logs -f docknode-pol-heimdall-1 --since 5m
docker logs -f docknode-pol-bor-1 --since 5m
docker compose down
```
