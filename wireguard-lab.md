# WireGuard VPN Lab — DigitalOcean + Docker

This GitHub Pages document covers my complete **WireGuard VPN deployment** using **Docker** on a **DigitalOcean droplet**, including configuration steps, client setup, NAT configuration, and traffic verification. All required screenshots, configuration files, and validation steps are included.

---

## 1. DigitalOcean Droplet Setup

I created a DigitalOcean account using the free 60-day credit and deployed a droplet with the following specifications:

- Operating System: Ubuntu 22.04 LTS  
- Plan: Basic ($6/month)  
- Resources: 1 GB RAM / 1 vCPU  
- Authentication: Root password login  
- Region: Closest available region  

I connected to the droplet using SSH:

ssh root@<DROPLET_PUBLIC_IP>

---

## 2. Docker and Docker Compose Installation

Docker and Docker Compose were installed on the droplet to support containerized deployment:

apt update  
apt install -y docker.io docker-compose  
systemctl enable --now docker  

Docker was verified to be running before continuing.

---

## 3. WireGuard Docker Setup

I created a working directory for WireGuard:

mkdir -p ~/wireguard  
cd ~/wireguard  

I then created a docker-compose.yml file with the following configuration:

version: "3.9"

services:
  wireguard:
    image: lscr.io/linuxserver/wireguard:latest
    container_name: wireguard
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    environment:
      - PUID=0
      - PGID=0
      - TZ=America/Chicago
      - SERVERURL=<DROPLET_PUBLIC_IP>
      - SERVERPORT=51820
      - PEERS=2
      - PEERDNS=1.1.1.1
      - INTERNAL_SUBNET=10.13.13.0
    volumes:
      - ./config:/config
      - /lib/modules:/lib/modules:ro
    ports:
      - 51820:51820/udp
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1
    restart: unless-stopped

The WireGuard container was started using:

docker-compose up -d

This automatically generated two peers:
- Peer 1: Mobile device  
- Peer 2: Desktop PC  

---

## 4. Enabling NAT on DigitalOcean

To allow VPN clients to route internet traffic through the droplet, Network Address Translation (NAT) and IP forwarding were enabled:

iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE  
echo "net.ipv4.ip_forward=1" | tee /etc/sysctl.d/99-wireguard.conf  
sysctl --system  
docker restart wireguard  

This configuration ensures VPN traffic is properly forwarded to the public internet.

---

## 5. Mobile Client Setup (Peer 1)

I exported the QR code for the mobile peer configuration:

docker cp wireguard:/config/peer1/peer1.png ./peer1.png  

The QR code was scanned using the WireGuard iOS application, completing the mobile VPN setup.

WireGuard iOS Client Connected:

<img width="309" height="654" alt="WireGuard iOS Connected" src="https://github.com/user-attachments/assets/18d571b8-61b8-46a1-8959-f7f9f473d461" />

---

## 6. Desktop Client Setup (Peer 2)

I exported the configuration file for the desktop peer:

docker cp wireguard:/config/peer2/peer2.conf ./peer2.conf  

This configuration was imported into the WireGuard Windows desktop application.

WireGuard Windows Client Connected:

<img width="725" height="631" alt="WireGuard Windows Connected" src="https://github.com/user-attachments/assets/cc398b24-6d86-4dbb-b42d-80fac15237aa" />

---

## 7. Public IP Verification

To confirm that traffic was routed through the VPN, I checked the public IP address using ifconfig.me before and after enabling the VPN on both devices:

- Phone – Before VPN  
- Phone – After VPN  
- PC – Before VPN  
- PC – After VPN  

In all cases, the public IP changed to match the DigitalOcean droplet, verifying successful VPN routing.

---

## 8. Repository Files Included

The following files are included in the repository:

- docker-compose.yml  
- index.md (this document)  
- Screenshots stored in the images/ directory  

---

## Summary

This lab demonstrates a complete **WireGuard VPN deployment** using Docker on a cloud-hosted Linux server. It reinforces practical skills in cloud infrastructure provisioning, containerized service deployment, secure remote access configuration, NAT routing, and traffic verification within a controlled and ethical environment.
