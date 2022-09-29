# Docker-Pi-hole

Configure Raspberry Pi MicroSD
In order to perform this operation, you will need the following equipments:

* A computer with ssh enabled
* A Raspberry Pi 3 or 4
* A MircroSD card (8GB min)
* A Wi-Fi connection

First, you need to install the OS on your Raspberry Pi.

On your computer, insert the MicroSD card, to install Raspbian OS on it.

You need to download Raspberry Pi Imager 
Download Raspberry PI OS Lite 64 (Bullseye).img
Release 04.04.2022 (For PI 3/4/400)

Launch Raspberry Pi Imager, then click on “Select OS” and select Raspberry Pi OS (64-bit) you just downloaded.

Press Cmd+Shift+X (on Mac) or Ctrl+Shift+X (on Windows) to get to the Advanced options window. 
Enable SSH and WiFi.

SSH into Raspberry Pi.

Update and Upgrade the Raspberry PI OS.

`$ sudo apt update && sudo apt full-upgrade -y`<br>
`$ sudo reboot`

Install extra tools.

`$ sudo apt install avahi-daemon git libffi-dev libssl-dev python3-dev python3 python3-pip dnsutils -y`<br>

Optional, create an alias.
`$ nano ~/.bashrc`

> alias ll='ls -alF'

Disable swap file usage by changing the parameter in file.

`$ sudo nano /etc/dphys-swapfile`

> CONF_SWAPSIZE=0

Add following to the end of the line.

`$ sudo nano /boot/cmdline.txt`

> cgroup_enable=cpuset cgroup_enable=memory cgroup_memory=1

Turn off Autologin / Console

`$ sudo raspi-config`

> Select "System option"<br>
> Select "Boot Auto Login"<br>
> Select "Console"<br>
> Reboot (Yes)


## Install Docker

`$ curl -sSL https://get.docker.com | sh`<br>
`$ sudo usermod -aG docker $USER`<br>
`$ sudo pip3 install docker-compose`<br>
`$ sudo reboot`

## Install Portainer

`$ docker run -d -p 8000:8000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest`

(docker run -d -p 9000:9000 --name=portainer -v /var/run/docker.sock:/var/run/docker.sock --restart=unless-stopped portainer/portainer:arm)

## Run Pi-hole and unbound with Docker-Compose
From the catalog ../docker-pihole-unbound/ run...

`$ docker-compose up -d to build and start pi-hole`

## Draft notes

mkdir pihole
cd pihole
docker-compose up -d
docker run --name=pihole -e TZ=Europe/Oslo -v pihole_app:/etc/pihole -v dns_config:/etc/dnsmasq.d -p 81:80 -p 53:53/tcp -p 53:53/udp --restart=unless-stopped pihole/pihole

# In container for setting new password
sudo pihole -a -p

# To run unbound from docker run
docker run --name=unbound \
--volume=/unbound:/opt/unbound/etc/unbound/ \
--publish=5335:53/tcp \
--publish=5335:53/udp \
--restart=unless-stopped \
--detach=true \
mvance/unbound-rpi:latest

================================================================================

To run Docker as a non-privileged user, consider setting up the
Docker daemon in rootless mode for your user:

    dockerd-rootless-setuptool.sh install

Visit https://docs.docker.com/go/rootless/ to learn about rootless mode.


To run the Docker daemon as a fully privileged service, but granting non-root
users access, refer to https://docs.docker.com/go/daemon-access/

WARNING: Access to the remote API on a privileged Docker daemon is equivalent
         to root access on the host. Refer to the 'Docker daemon attack surface'
         documentation for details: https://docs.docker.com/go/attack-surface/

================================================================================
