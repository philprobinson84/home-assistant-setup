# Home Assistant Configuration for Boxtree Cottage

## Server


## Room Clients

A RPi is located in each room, connected to an HTU31D temperature and humidity sensor and a speaker.

### Install Raspberry Pi OS (Bookworm)

Flash the SD card, pre-configure with Wifi info, and set hostname to rpi-<room>.

### Setup HTU31D

```bash
mkdir projects
cd projects
```

Follow the instructions in the readme here: https://github.com/philprobinson84/home-assistant-htu31d

### Setup Airplay

We will use the Docker image.

Bookworm uses the PipeWire audio backend instead of PulseAudio, but we target the underlying ALSA device.

#### Install Docker

```bash
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done

# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

# Install docker:
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Setup non-root usage:
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
```

#### Run shairport-sync container

```bash
cd shairport-sync
docker compose up -d
```

## Whole-Home Audio

One of the RPi clients is also configured to run owntone, and a special shairport-sync configuration to map a single AirPlay target to multiple rooms.

### Create pipes for the containers

```bash
mkfifo ~/home-assistant-setup/owntone/media/shairport
mkfifo ~/home-assistant-setup/owntone/media/shairport.metadata
```

### Run the shairport-sync container for owntone

Check IP settings in the compose file.

```bash
cd shairport-sync-owntone
docker compose up -d
```

### Run the owntone container

```bash
cd owntone
docker compose up -d
```

### Monitoring - Dozzle

One machine as server:

```bash
cd dozzle
docker compose up -d
```

On the rest:

```bash
cd dozzle-agent
docker compose up -d
```

### Reference

Permissions for the pipes on old setup:

```bash
pi@rpi-hall:~ $ ls -l /srv/music/shairport
prw-rw-rw- 1 root root 0 Mar 20 22:15 /srv/music/shairport
pi@rpi-hall:~ $ ls -l /srv/music/shairport.metadata
prw-rw-rw- 1 root root 0 Mar 20 22:15 /srv/music/shairport.metadata
pi@rpi-hall:~ $ stat /srv/music/shairport
  File: /srv/music/shairport
  Size: 0               Blocks: 0          IO Block: 4096   fifo
Device: b302h/45826d    Inode: 522331      Links: 1
Access: (0666/prw-rw-rw-)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2020-12-28 12:39:22.913494036 +0000
Modify: 2025-03-20 22:15:46.679625605 +0000
Change: 2025-03-20 22:15:46.679625605 +0000
 Birth: -
pi@rpi-hall:~ $ stat /srv/music/shairport.metadata
  File: /srv/music/shairport.metadata
  Size: 0               Blocks: 0          IO Block: 4096   fifo
Device: b302h/45826d    Inode: 522713      Links: 1
Access: (0666/prw-rw-rw-)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2020-12-28 12:39:44.023175289 +0000
Modify: 2025-03-20 22:15:46.689625468 +0000
Change: 2025-03-20 22:15:46.689625468 +0000
 Birth: -
pi@rpi-hall:~ $
```