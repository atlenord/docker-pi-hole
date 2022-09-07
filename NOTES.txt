##### NOTES #####

Based on Raspberry PI OS Lite 64 (Bullseye).img
Release 04.04.2022 (For PI 3/4/400)


sudo apt-get update && sudo apt-get upgrade -y
(sudo apt full-upgrade)

sudo reboot
nano ~/.bashrc
- alias ll='ls -alF'
curl -sSL https://get.docker.com | sh
sudo usermod -aG docker $USER
sudo apt install avahi-daemon -y
sudo reboot
sudo apt install git -y
sudo apt install libffi-dev libssl-dev -y
sudo apt install python3-dev -y
sudo apt install python3 python3-pip -y
sudo pip3 install docker-compose
sudo apt install dnsutils -y
sudo raspi-config # Turn off Autologin / Console
- Select "System option"
- Select "Boot Auto Login"
- Select "Console"

(docker run -d -p 9000:9000 --name=portainer -v /var/run/docker.sock:/var/run/docker.sock --restart=unless-stopped portainer/portainer:arm)
docker run -d -p 8000:8000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest
mkdir pihole
cd pihole
docker-compose up -d
docker run --name=pihole -e TZ=Europe/Oslo -v pihole_app:/etc/pihole -v dns_config:/etc/dnsmasq.d -p 81:80 -p 53:53/tcp -p 53:53/udp --restart=unless-stopped pihole/pihole

# In container for setting new password
sudo pihole -a -p

docker run --name=unbound \
--volume=/unbound:/opt/unbound/etc/unbound/ \
--publish=5335:53/tcp \
--publish=5335:53/udp \
--restart=unless-stopped \
--detach=true \
mvance/unbound-rpi:latest

---
#DOCKER-COMPOSE template in Portainer
version: "3"
services:
  unbound:
    container_name: unbound-dns
    hostname: unbound-dns
    image: mvance/unbound-rpi:latest
    networks:
      my_pi_network:
        ipv4_address: 10.0.0.11
    cap_add:
      - NET_ADMIN
    restart: unless-stopped
  pihole:
    container_name: pi-hole
    hostname: pi-hole
    image: pihole/pihole:latest
    networks:
      my_pi_network:
        ipv4_address: 10.0.0.10
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "80:80/tcp"
    environment:
      - TZ=Europe/Oslo
      - WEBPASSWORD=${WEB_PASSWORD}
      - DNS1=10.0.0.11#53
      - DNS2=no
    volumes:
      - etc-pihole:/etc/pihole
      - etc-dnsmasq.d:/etc/dnsmasq.d
    cap_add:
      - NET_ADMIN
    restart: unless-stopped
volumes:
  etc-pihole:
  etc-dnsmasq.d:

networks:
  my_pi_network:
    driver: bridge
    ipam:
      config:
        - subnet: 10.0.0.0/16
          gateway: 10.0.0.1
---

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
