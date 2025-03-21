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

