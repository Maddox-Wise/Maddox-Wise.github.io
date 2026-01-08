# Docker + Docker Compose Documentation  
### Pi-hole, Unbound, and Nextcloud Stack on My Homelab Server

**Author:** Maddox Wise  
**Course:** CYB 3353 – System Administration  
**Assignment:** Docker Application Deployment  
**Date:** 11-16-25  

---

## 1. Overview

For this assignment, I deployed a Docker-based application stack on my homelab Linux server using **Docker** and **Docker Compose**. I selected applications that I actively use in my personal homelab environment:

- **Unbound** – A validating, recursive DNS resolver  
- **Pi-hole** – A network-wide DNS sinkhole and ad blocker  
- **Nextcloud** – A self-hosted cloud storage and productivity platform  

I created a `docker-compose.yml` file, organized persistent data directories, and verified that each service runs successfully. This documentation is designed so another student could recreate the same setup on their own system.

---

## 2. System Environment

- **Host OS:** Ubuntu Server (Homelab VM)  
- **Architecture:** x86_64  
- **Working Directory:** `/home/maddox/`  
- **Files Created:** `docker-compose.yml` and persistent volume directories  

---

## 3. Installing Docker & Docker Compose

Docker and Docker Compose were already installed on my homelab server. For reference, a typical installation process is shown below:

```bash
sudo apt update
sudo apt install docker.io docker-compose-plugin -y
sudo systemctl enable docker --now
Verify Installation
bash

docker --version
docker compose version
4. Folder Structure
Before deploying containers, I created directories to store persistent configuration and data:

text

/home/maddox/
├── unbound/
├── pihole/
│   ├── etc-pihole/
│   └── etc-dnsmasq.d/
└── nextcloud/
    ├── config/
    └── data/
These folders ensure container data persists across updates and reboots.

5. Docker Compose File
Below is the exact docker-compose.yml file used for this deployment:

yaml

version: "3.9"

services:
  ###############################################################
  # UNBOUND – Local DNS Resolver
  ###############################################################
  unbound:
    image: mvance/unbound:latest
    container_name: unbound
    restart: unless-stopped
    ports:
      - "5335:53/udp"
    volumes:
      - /home/maddox/unbound:/etc/unbound

  ###############################################################
  # PI-HOLE – Network Ad Blocking
  ###############################################################
  pihole:
    image: pihole/pihole:latest
    container_name: pihole
    restart: unless-stopped
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "8081:80"
    environment:
      - TZ=America/Chicago
      - WEBPASSWORD=changeme
      - DNS1=127.0.0.1#5335
      - DNS2=1.1.1.1
    volumes:
      - /home/maddox/pihole/etc-pihole:/etc/pihole
      - /home/maddox/pihole/etc-dnsmasq.d:/etc/dnsmasq.d
    depends_on:
      - unbound
    cap_add:
      - NET_ADMIN

  ###############################################################
  # NEXTCLOUD – Self-Hosted Cloud Storage
  ###############################################################
  nextcloud:
    image: lscr.io/linuxserver/nextcloud:latest
    container_name: nextcloud
    restart: unless-stopped
    ports:
      - "8080:443"
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Chicago
    volumes:
      - /home/maddox/nextcloud/config:/config
      - /home/maddox/nextcloud/data:/data

networks:
  default:
    driver: bridge
6. Deploying the Stack
Navigate to the directory containing docker-compose.yml and start the stack:

bash

cd /home/maddox
docker compose up -d
The -d flag runs containers in the background

Docker automatically pulls images and starts all services

Check Running Containers
bash

docker ps
7. Service Verification
Unbound
Runs a recursive DNS resolver on port 5335

Test command:

bash

dig @127.0.0.1 -p 5335 google.com
Pi-hole
Web interface available at:
http://<server-ip>:8081/admin

Confirmed Pi-hole uses Unbound as its upstream DNS

Logged in using the password set in the environment variables

Nextcloud
Web interface available at:
https://<server-ip>:8080

Completed initial setup through the web UI

Verified file uploads, users, and apps function correctly

8. Managing the Stack
Stop All Services
bash

docker compose down
Restart Services
bash

docker compose restart
View Logs
bash

docker compose logs -f
Update Containers
bash

docker compose pull
docker compose up -d
9. Troubleshooting Notes
Port 53 conflicts:
Pi-hole may fail if another DNS service is running.
Solution: Disable systemd-resolved or change ports.

Nextcloud port conflicts:
Use an alternate mapping such as 8443:443.

Volume permission issues:
Fixed using:

bash

sudo chown -R $USER:$USER /home/maddox/*
Unbound not resolving:
Typically caused by invalid configuration files. Restarted the container and verified configs.

10. Files Included in Submission
docker-compose.yml


